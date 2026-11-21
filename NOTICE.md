# NOTICE

## 上游项目

本项目的前端部分（`frontend/`）派生自：

- **deepseek-ai/chat-demo** — <https://github.com/deepseek-ai/chat-demo>
- 版本：0.1.3
- 许可证：MIT
- 版权：Copyright (c) DeepSeek

上游 MIT 许可证全文见 `frontend/LICENSE`，并在 `LICENSE` 中一并保留。

## 修改说明

根据 MIT 许可证的要求，以下列出相对于上游的主要修改。修改时间：2026 年 11 月。

**新增功能**

- 会话级「角色设定」（`src/components/session/SystemPromptDialog.tsx`、`systemPromptDialogState.ts`），并通过前置问答注入上下文（`src/services/chat/reply.ts` 的 `toRequestMessages`）
- 每轮 token 用量与上下文占用显示（`src/components/messages/AssistantMessage.tsx`）
- 服务端文件管理（`src/components/settings/FileManagerDialog.tsx`）与本地附件管理（`LocalFilesDialog.tsx`、`src/services/chat/localFiles.ts`）
- 图片全屏查看与手势缩放（`src/components/files/ImageViewer.tsx`）
- 导入/导出进度条（`src/components/settings/SettingsDialog.tsx`、`zipReader.ts`、`importArchive.ts`、`exportArchive.ts`）
- 推理强度设置（`src/services/settings.ts`、`src/services/chat/store.ts`）
- 清除缓存入口
- 拍照上传的页面侧桥接（`src/components/chatInput/ChatInput.tsx`）

**行为调整**

- 视图层动画：消息、弹窗、菜单、抽屉过渡，并支持 `prefers-reduced-motion`
- 禁用页面级缩放（`src/index.html` 的 viewport meta），图片查看器内自行实现缩放
- 录音采样率降级重采样（`src/services/speech/recorder.ts`）
- 麦克风约束失败时降级重试（同上）
- 应用名称与界面文案调整

**未修改**

模型请求的协议细节、附件上传流程、历史归档格式（`export_format_version: 3`）均保持与上游一致，以保证与官方导出的数据互通。

## 商标

本仓库不包含深度求索的任何商标、Logo 或品牌素材。应用图标为自制矢量图形。

「DeepSeek」「深度求索」是其权利人的商标。本项目名称与描述中对 DeepSeek 的提及仅用于说明兼容性，不构成任何关联、赞助或背书关系。
