# 逗子制片所 CLI 使用说明

本接口复用桌面应用的真实项目库、业务状态和媒体服务。命令会启动一个不可见 Electron 窗口，等待业务初始化、执行操作、保存活动项目，返回 JSON 后退出。它不是独立的云服务，也不是对软件界面的鼠标模拟。

本次实现保存在本地，未经用户验收授权不得上传 GitHub。旧的 Mac 安装包不含本接口；请使用包含 CLI 的新构建。

## 1. 环境与入口

需要 Node.js 20 或更新版本，以及能在当前操作系统运行的 CLI 版桌面程序。Mac 的签名、公证要求仍适用；CLI 不会绕过系统安全检查。Windows 包需要 Windows x64 电脑运行；在 Mac 上不能执行 Windows EXE。

开发目录：

```sh
pnpm install
pnpm build
node scripts/douzi-cli.mjs --help
node scripts/douzi-cli.mjs capabilities
```

已解压 Windows 应用（PowerShell）：

```powershell
node .\douzi-cli.mjs capabilities --app ".\逗子制片所.exe" --data-dir "C:\Douzi-Test\cli-data"
```

安装版/源码目录中脚本位于 `resources/app/scripts/douzi-cli.mjs` 或 `scripts/douzi-cli.mjs`。也可从测试交付目录使用同名独立脚本，并以 `--app` 指定 EXE。脚本不需要 npm 安装依赖即可连接指定程序，但该程序必须是包含 CLI 的版本。

Mac 的 `--app` 可以传 `.app` 目录，也可传包内真实可执行文件。没有 `--app` 时从源码目录使用本地 electron 依赖启动。

## 2. 数据目录和并发

默认数据目录为用户主目录的 `.douzi-studio-cli`，与 GUI 的默认项目库隔离。**它是 Electron 用户数据根目录，不是其下的 projects 子目录。**

请为同一工作流始终使用相同的 `--data-dir`。例如本文命令省略该参数时，全部使用默认 CLI 项目库。接入原桌面项目库前先退出桌面应用并备份数据。CLI 不负责与正在打开的 GUI 实例合并修改。

同一 CLI 数据目录使用 `douzi-cli.lock` 排他锁，第二个并发命令会报 busy。如果系统异常退出留下锁，先确认该目录没有 CLI 进程使用，再人工移走该锁文件。不要为解除锁去删除 projects 目录。

所有导出使用独占创建，不覆盖已存在的目标文件。请指定新文件名。二进制文件不会直接写到终端。

## 3. 通用参数与输出

| 参数 | 含义 |
| --- | --- |
| `--app PATH` | 当前平台的 CLI 版应用可执行文件或 Mac `.app` |
| `--data-dir PATH` | Electron 用户数据根目录 |
| `--project ID` | 执行前打开此项目；项目 ID 从创建结果或 projects 查询获得 |
| `--args JSON` | 业务方法的位置参数数组，例如 `["id",{"prompt":"..."}]` |
| `--args @FILE.json` | 从 UTF-8 JSON 文件读取位置参数，推荐用于 Windows 和复杂参数 |
| `--file PATH` | 项目 ZIP 导入文件 |
| `--out PATH` | ZIP、二进制 API 或单个浏览器下载文件的输出位置 |
| `--out-dir PATH` | 业务动作产生多个下载文件时的输出文件夹 |
| `--method METHOD` | api 的 HTTP 方法，默认 GET |
| `--body JSON` / `--body @FILE.json` | API JSON 请求体 |
| `--timeout SECONDS` | 等待本地操作的上限，默认 300 秒 |
| `--allow-spend` | 显式允许生成/供应商动作及非 GET API；可能产生外部调用和费用 |

正常输出示例：

```json
{"ok":true,"result":{"result":null,"activeProjectId":"项目ID","message":"..."},"downloads":[]}
```

`state`、`projects`、`inspect` 返回其对象作为外层 `result`。`call` 返回的业务方法值位于 `result.result`。异常输出到 stderr，例如 `{"ok":false,"error":"节点不存在"}`，进程返回非零。帮助输出为纯文本。

`ok:true` 表示这次本地方法执行返回，不保证云端已经生成成功。业务层有些动作会捕获供应商错误并把信息写入 `lastMessage`、`importError` 或任务 `status/error`；调用后需要用 inspect 或 state 检查业务结果。超时后远程任务可能继续运行，请先查询任务，避免重复提交付费生成。

## 4. 查询能力与项目管理

```sh
node scripts/douzi-cli.mjs capabilities
node scripts/douzi-cli.mjs projects
node scripts/douzi-cli.mjs call library createProject --args '["CLI 画布","","canvas"]'
node scripts/douzi-cli.mjs call library createProject --args '["CLI 短剧","剧本正文","short_drama"]'
node scripts/douzi-cli.mjs state --project PROJECT_ID
node scripts/douzi-cli.mjs inspect --project PROJECT_ID
node scripts/douzi-cli.mjs call library renameProject --args '["PROJECT_ID","新名称"]'
node scripts/douzi-cli.mjs call library duplicateProject --args '["PROJECT_ID"]'
node scripts/douzi-cli.mjs call library archiveProject --args '["PROJECT_ID",true]'
node scripts/douzi-cli.mjs call library archiveProject --args '["PROJECT_ID",false]'
node scripts/douzi-cli.mjs call library trashProject --args '["PROJECT_ID"]'
node scripts/douzi-cli.mjs call library restoreProject --args '["PROJECT_ID"]'
```

`deleteForever` 为永久删除动作，仅对明确需要永久删除的项目调用。它与移入废纸篓不同。完整方法签名见 [CLI-ACTIONS.md](CLI-ACTIONS.md)。

## 5. 无限画布

下列命令需指定 canvas 类型项目。添加和连接操作返回新节点/连线 ID：

```sh
node scripts/douzi-cli.mjs canvas add-note --project PROJECT_ID --args '[{"text":"创意便笺","position":{"x":80,"y":80}}]'
node scripts/douzi-cli.mjs canvas update --project PROJECT_ID --args '["NODE_ID",{"text":"修改内容","position":{"x":420,"y":80}}]'
node scripts/douzi-cli.mjs canvas connect --project PROJECT_ID --args '["NODE_A","NODE_B"]'
node scripts/douzi-cli.mjs canvas viewport --project PROJECT_ID --args '[{"x":0,"y":0,"zoom":1}]'
node scripts/douzi-cli.mjs canvas remove --project PROJECT_ID --args '["EDGE_OR_NODE_ID"]'
```

移除节点同时移除与它相关的连线。移除媒体画布节点不会自动永久删除磁盘素材。CLI 画布操作直接修改项目文档，不增加旧短剧工作台的 undo 历史；需要回退时先导出完整项目包。

文件通过 `$file` 描述：

```json
[{"$file":"C:/Users/you/Desktop/reference.png"}]
```

保存为 `media-args.json` 后执行：

```sh
node scripts/douzi-cli.mjs canvas import-media --project PROJECT_ID --args @media-args.json
```

重关联参数为 `["MEDIA_NODE_ID", {"$file":"替换文件绝对路径"}]`，调用 `canvas relink`。它保留画布节点 ID 和位置，因此原连线可继续引用此节点。

图片、视频、音频、附件都支持。CLI 根据扩展名推断常见 MIME；特殊类型可用 `{"$file":"...","type":"具体 MIME"}`。文件经本地临时请求转为 File 后进入现有持久化业务；单文件上限沿用 512 MB。Base64 传递需要额外内存，大文件不保证在低内存机器可用。

## 6. 完整项目包与快照

```sh
node scripts/douzi-cli.mjs package export --project PROJECT_ID --out ./project-backup.zip
node scripts/douzi-cli.mjs package import --project TARGET_ID --file ./project-backup.zip
node scripts/douzi-cli.mjs state --project PROJECT_ID
```

ZIP 导入替换目标项目内容，请先创建空白目标或备份。包含媒体的导入会把文件持久保存到目标项目自己的 assets 目录。`state` 仅输出 JSON 快照，不包含本地媒体字节；跨电脑共享请选择完整 ZIP。

`call project replaceProjectSnapshot --args @snapshot-args.json --project ID` 可替换整个快照，文件内容必须为 `[快照对象]`，不是裸快照。复杂节点和业务类型请参考 domain/projectLibrary.ts、domain/project.ts 及实际 state 输出。

## 7. 剧本、制片蓝图和导演流程

```sh
node scripts/douzi-cli.mjs call project importScript --project PROJECT_ID --args '["剧本正文","标题"]'
node scripts/douzi-cli.mjs inspect --project PROJECT_ID
node scripts/douzi-cli.mjs call project confirmPendingImport --project PROJECT_ID --args '[]'
node scripts/douzi-cli.mjs call project setDirectorCommand --project PROJECT_ID --args '["调整当前制作计划"]'
node scripts/douzi-cli.mjs call project startDirectorPlan --project PROJECT_ID --args '[]'
```

`importScript` 是本地拆分入口。`importScriptWithAgnes` 调用现有 AI 剧本业务，需 `--allow-spend`。`updatePendingBlueprintSegment/removePendingBlueprintSegment/mergePendingBlueprintSegments` 用于确认前调整制作段；随后 `confirmPendingImport` 确认。

注意：pendingImport、撤销历史等临时状态未必进入持久化快照。需要连续执行“导入—查看/修改—确认”时，使用下文 batch；不要假设每个临时字段会跨进程保存。

导演的 setDirectorCommand、setDirectorTaskFocus、advanceDirectorPlan、runDirectorStep、pauseDirectorForReview、recordVisualCheck 均通过 `call project` 调用。CLI 不替你作出创作审核结论，确认和审核动作需调用者明确选择。

## 8. 素材、镜头、故事板、视频和语音

以下功能直接调用当前业务状态动作，参数完整签名见动作参考：

| 领域 | 主要方法 |
| --- | --- |
| 资产信息 | renameAsset、changeAssetKind、updateAssetRequirement、updateAssetVersionStatus |
| 素材文件 | replaceAssetMedia、addImportedImages（参数中的 File / File[] 用 `$file`） |
| 云端引用 | updateAssetPublicUrl、updateAssetCloudReference、setAutoCompleteCloudReferences |
| 镜头 | updateShotData、focusShotForProduction、applyDirectorStyleCard |
| 关联关系 | addConnection、removeEdge、updateEdgeKind、updateNodePosition |
| 生成任务 | createGenerationForShot、createGenerationForExecutionUnit、updateGenerationSettings、deleteGenerationTask |
| 执行计划 | prepareExecutionPlan、reviewExecutionPlan、updateExecutionPrompt |
| 分格故事板 | updatePanelPrompt、updatePanelCompositionReference、generateStoryboardPanel、adoptStoryboardPanel、assembleStoryboardPanels |
| 首尾帧/参考素材 | setGenerationInputAsset、setGenerationReferenceAssets、captureExecutionEnding |
| 生成执行 | runGenerationTask、generateAssetImage、runAgnes、runImageModel、runVolcengineSeedance、runExternalVideo |
| 修复 | prepareActionRepair、applyActionRepair、dismissActionRepair、regenerateVideoWithAudio |
| TTS | updateTTSSettings、generateTTSForShot、generateTTSForAllShots |
| 本地预览与合成 | runFullLocalPreview、runCompositeExport |

例：

```sh
node scripts/douzi-cli.mjs call project updateShotData --project PROJECT_ID --args '["SHOT_ID",{"title":"新镜头标题","durationSeconds":5}]'
node scripts/douzi-cli.mjs call project updateAssetRequirement --project PROJECT_ID --args '["ASSET_NODE_ID",{"prompt":"修改后的提示词"}]'
node scripts/douzi-cli.mjs call project runGenerationTask --project PROJECT_ID --args '["TASK_NODE_ID"]' --allow-spend --timeout 900
node scripts/douzi-cli.mjs inspect --project PROJECT_ID
node scripts/douzi-cli.mjs call project runCompositeExport --project PROJECT_ID --args '[]' --allow-spend --timeout 900 --out ./preview.webm
```

`createGenerationForShot` 只创建任务，不等于生成完成；返回的任务 ID 再用于执行。生成是否可用仍受 API 配置、服务额度、模型输入约束、审核门槛与实际接口状态限制。CLI 不跳过现有制片流程中的守门规则。

导出动作若触发浏览器下载，后台会收集下载结果；单文件用 `--out`，多个文件用 `--out-dir`。指定的文件已存在时不会覆盖。某些方法仅创建资产引用而不触发下载，应从 state/inspect 查询最终 URL 或路径。

## 9. API 配置、诊断和完整后端入口

```sh
node scripts/douzi-cli.mjs api /api/config/status
node scripts/douzi-cli.mjs api /api/convert/check-ffmpeg
node scripts/douzi-cli.mjs api /api/config/provider --method POST --body @provider.json --allow-spend
node scripts/douzi-cli.mjs api /api/config/test-provider --method POST --body '{"provider":"供应商 ID"}' --allow-spend
```

`provider.json` 内容遵循现有 `/api/config/provider`：provider、apiKey，以及可选 baseUrl、modelId、imageModelId、textModelId、videoModelId、visionModelId、adapter、upload。请使用专用本地文件传递密钥，避免把它放入共享命令记录或文档。不要把 provider.json 发给别人。

API 只允许当前应用同源的 `/api/` 路径。GET 可直接执行；非 GET 必须显式 `--allow-spend`（配置写入虽然通常不产生费用，也采用同一显式写入参数）。JSON 返回 JSON，text 返回文本，其他二进制返回需指定 `--out`。

完整路由索引见 CLI-ACTIONS.md。它覆盖各供应商图片/视频/语音、任务查询、云端素材、诊断等入口；每个路由的 body 以 server/agnes-proxy.mjs 为准。不是通用外部网址下载工具，也不接受任意主机地址。

## 10. batch 同进程工作流

`batch --args @workflow.json --project ID` 顺序执行多个命令对象。示例：

```json
[
  {"command":"call","scope":"project","method":"importScript","args":["剧本正文","测试短剧"]},
  {"command":"inspect"},
  {"command":"call","scope":"project","method":"confirmPendingImport","args":[]},
  {"command":"state"}
]
```

```sh
node scripts/douzi-cli.mjs batch --project PROJECT_ID --args @workflow.json
```

子命令也使用本文的内部 command 名：`call`、`canvas`、`state`、`inspect`、`projects`、`api`、`package-import`、`package-export`。文件标记同样支持；但批量导出的 `$binary` 结果不会自动保存多个文件，项目包导出请单独执行。batch 不是数据库事务：中间失败不会回滚之前已保存的修改。`--allow-spend` 从外层显式传递，子对象不能自行扩大权限。

撤销/重做仅针对业务层维护的同进程历史有效。单独运行 undo 命令时，不会恢复另一个已经退出进程的内存历史。导入剧本、蓝图检查和确认推荐 batch；修改可持久化字段可使用多次普通命令。

## 11. 验收与限制

- capabilities 动态列出当前构建全部 project/library 公开动作，避免文档更新晚于实现；完整签名索引在 CLI-ACTIONS.md。
- CLI 不模拟系统安全弹窗、安装、公证、窗口排版、鼠标拖动等行为，这些仍需 GUI/系统实机验收。
- API 调用需要用户自己的配置和额度；未把所有供应商的付费生成接口当成已真实验收。
- Windows 交叉打包不等于 Windows 实机运行通过；Windows 版及其 CLI 的运行验收状态以交付记录为准。
- 默认独立数据目录、排他锁和禁止输出覆盖可以降低误操作，但永久删除、重置快照和付费生成仍由调用者负责明确选择。
- 尚未完成 Apple 签名/公证的 Mac 版本不能因为加入 CLI 就变成可对客户正常分发的版本。
