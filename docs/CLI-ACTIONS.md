# CLI 完整业务动作参考

此清单从当前源码接口提取。运行 `capabilities` 可获取当前构建实际支持的动作。参数按签名位置放入 `--args` JSON 数组。复杂类型请参见相应源码定义；File 用 `{ "$file": "文件路径" }` 传入。

## 项目业务动作

调用：`call project 方法名 --args JSON --project 项目ID`。`Promise` 表示命令等待该动作的 Promise 完成；不保证云端任务本身已经完成。

```typescript
  prepareActionRepair: (segmentId: string, provider?: string) => Promise<boolean>;
  applyActionRepair: () => boolean;
  dismissActionRepair: () => void;
  importPackage: (input: unknown) => void;
  importProjectPackageZip: (file: File, projectId?: string) => Promise<boolean>;
  importScript: (script: string, title?: string, options?: ScriptImportOptions) => void;
  importScriptWithAgnes: (script: string, title?: string, options?: ScriptImportOptions) => Promise<void>;
  updatePendingBlueprintSegment: (segmentId: string, values: Partial<Pick<ProductionSegment, "title" | "firstFrameMode" | "storyboardMode" | "videoMode">>) => void;
  removePendingBlueprintSegment: (segmentId: string) => void;
  mergePendingBlueprintSegments: (firstId: string, secondId: string) => void;
  confirmPendingImport: (preferredShotId?: string, destination?: ProductionDestination) => void;
  focusShotForProduction: (shotId: string, destination?: ProductionDestination, taskKind?: ShotWorkTaskKind) => void;
  discardPendingImport: () => void;
  loadSample: (input: unknown) => void;
  selectNode: (nodeId: string | null) => void;
  selectShot: (shotId: string | null) => void;
  selectEdge: (edgeId: string | null) => void;
  openPreview: (preview: { url: string; title: string; kind: "image" | "video" }) => void;
  closePreview: () => void;
  setViewMode: (mode: ProjectState["viewMode"]) => void;
  setAutoCompleteCloudReferences: (enabled: boolean) => void;
  addConnection: (source: string, target: string) => void;
  removeEdge: (edgeId: string) => void;
  updateNodePosition: (nodeId: string, position: { x: number; y: number }) => void;
  setGenerationInputAsset: (generationNodeId: string, assetNodeId: string, edgeKind: "first_frame" | "last_frame" | "storyboard_control") => void;
  setGenerationReferenceAssets: (generationNodeId: string, assetNodeIds: string[]) => void;
  updateEdgeKind: (edgeId: string, kind: CanvasEdgeKind) => void;
  updatePromptText: (nodeId: string, text: string) => void;
  updateShotData: (shotId: string, values: Partial<Pick<ShotNodeData, "title" | "location" | "characters" | "durationSeconds" | "narration">>) => void;
  applyDirectorStyleCard: (cardId: string, shotId?: string | null) => void;
  updateAssetPublicUrl: (nodeId: string, publicUrl: string) => void;
  updateAssetCloudReference: (nodeId: string, values: Partial<Pick<AssetNodeData, "publicUrl" | "fileId" | "assetId" | "cloudProvider" | "cloudStatus" | "cloudMessage" | "cloudExpiresAt" | "cloudPreparedAt">>) => void;
  updateAssetVersionStatus: (nodeId: string, status: NonNullable<AssetNodeData["versionStatus"]>, note?: string) => void;
  renameAsset: (nodeId: string, label: string) => void;
  updateAssetRequirement: (nodeId: string, values: Partial<Pick<AssetNodeData, "description" | "prompt" | "negativePrompt">>) => void;
  changeAssetKind: (nodeId: string, kind: AssetKind) => void;
  replaceAssetMedia: (nodeId: string, file: File) => void;
  addImportedImages: (files: File[], kind: AssetKind) => void;
  createGenerationForShot: (shotId: string, taskKind: GenerationTaskKind, videoInputMode?: VideoInputMode, outputAssetKind?: AssetKind, promptOverride?: string, productionScope?: { id: string; shotIds: string[] }, execution?: ExecutionBinding) => string;
  createGenerationForExecutionUnit: (planId: string, unitId: string, taskKind: GenerationTaskKind) => string;
  updateExecutionPrompt: (planId: string, revision: number, unitId: string, taskKind: 'storyboard_image' | 'storyboard_control_video', prompt: string) => void;
  updatePanelPrompt: (planId: string, unitId: string, index: number, prompt: string) => void;
  updatePanelCompositionReference: (planId: string, unitId: string, index: number, image?: string) => void;
  generateStoryboardPanel: (planId: string, unitId: string, index: number, modelId: ModelId) => Promise<void>;
  adoptStoryboardPanel: (assetId: string) => void;
  assembleStoryboardPanels: (planId: string, unitId: string) => Promise<void>;
  reviewExecutionPlan: (planId: string) => boolean;
  prepareExecutionPlan: (segmentId: string, modelId: ModelId, provider?: string) => Promise<boolean>;
  captureExecutionEnding: (videoAssetId: string) => Promise<string>;
  deleteGenerationTask: (nodeId: string) => void;
  deleteAssetNode: (nodeId: string) => void;
  updateGenerationSettings: (nodeId: string, values: Partial<Pick<GenerationNodeData, "modelId" | "qualityTier" | "videoStrategy" | "videoInputMode" | "whiteModel" | "generateAudio" | "audioRefUrls" | "prompt" | "negativePrompt">>) => void;
  runGenerationTask: (nodeId: string) => Promise<void>;
  runFullLocalPreview: () => Promise<void>;
  runCompositeExport: () => Promise<void>;
  updateTTSSettings: (settings: Partial<ProjectState["ttsSettings"]>) => void;
  generateTTSForShot: (shotId: string) => Promise<void>;
  generateTTSForAllShots: () => Promise<void>;
  startDirectorPlan: () => void;
  setDirectorCommand: (command: string) => void;
  setDirectorTaskFocus: (task?: DirectorTaskFocus) => void;
  advanceDirectorPlan: () => void;
  runDirectorStep: () => Promise<void>;
  pauseDirectorForReview: (reason?: string) => void;
  recordVisualCheck: (record: VisualCheckRecord) => void;
  runAgnes: (nodeId: string) => Promise<void>;
  runImageModel: (nodeId: string) => Promise<void>;
  generateAssetImage: (nodeId: string, modelId?: ModelId) => Promise<void>;
  runVolcengineSeedance: (nodeId: string) => Promise<void>;
  runExternalVideo: (nodeId: string) => Promise<void>;
  regenerateVideoWithAudio: (nodeId: string) => Promise<void>;
  undo: () => void;
  redo: () => void;
  resetProject: () => void;
  getProjectSnapshot: () => ProjectSnapshot;
  replaceProjectSnapshot: (snapshot: ProjectSnapshot) => void;
```

## 项目库业务动作

调用：`call library 方法名 --args JSON`。项目列表使用 `projects`，活动项目快照使用 `state --project ID`，临时计划与任务细节使用 `inspect --project ID`。

```typescript
  initializeLibrary(): Promise<void>;
  refresh(): Promise<void>;
  openProject(id: string): Promise<void>;
  createProject(title?: string, scriptText?: string, projectType?: ProjectType): Promise<void>;
  importProject(snapshot: ProjectSnapshot): Promise<void>;
  saveCurrentProject(): Promise<void>;
  returnToLibrary(): Promise<void>;
  renameProject(id: string, title: string): Promise<void>;
  duplicateProject(id: string): Promise<void>;
  archiveProject(id: string, archived: boolean): Promise<void>;
  trashProject(id: string): Promise<void>;
  restoreProject(id: string): Promise<void>;
  deleteForever(id: string): Promise<void>;
```

## 后端 API 路由索引

调用：`api 路径 --method GET|POST --body JSON`。这是源码中出现的静态路径索引，包含内部调用路径；HTTP 方法和 body 需按服务端分支确定，不能把每个路径都当成 GET。POST 必须添加 `--allow-spend`。动态任务 ID、查询参数按实际接口填写。

- `/api/agent/action-repair`
- `/api/agent/execution-plan`
- `/api/agent/panel-repair`
- `/api/agent/visual-inspection`
- `/api/agnes/image`
- `/api/agnes/script-package`
- `/api/agnes/status`
- `/api/agnes/video`
- `/api/agnes/video-result`
- `/api/assets/prepare-cloud-reference`
- `/api/config/clear-provider`
- `/api/config/provider`
- `/api/config/status`
- `/api/config/test-provider`
- `/api/convert/check-ffmpeg`
- `/api/convert/mp4`
- `/api/gemini/image`
- `/api/kling/video`
- `/api/kling/video-task`
- `/api/media/cache-image`
- `/api/runway/video`
- `/api/runway/video-task`
- `/api/tts/clear`
- `/api/tts/config`
- `/api/tts/status`
- `/api/tts/synthesize`
- `/api/tts/test`
- `/api/voice-clone/status`
- `/api/voice-clone/upload`
- `/api/volcengine/image`
- `/api/volcengine/seedance/task`
- `/api/volcengine/seedance/video`

## 覆盖边界

本参考覆盖 project/library 公开业务动作与代理 API 路由入口。鼠标悬停、窗口布局、文件选择对话框、系统安全提示等 GUI/系统行为不通过 CLI 模拟。视图/选中/预览等瞬时动作可在 batch 内调用，但后台窗口不可见，退出后不会保留屏幕效果。未声称每个付费供应商接口均已真实调用通过。


