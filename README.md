# 深聊 · DeepChat

简体中文 | [English](README.en.md)

一个基于 [DeepSeek 官方 chat-demo](https://github.com/deepseek-ai/chat-demo) 改造的 **Android 客户端**，把官方那套聊天前端装进一个原生壳里，并补上了一些日常使用需要的能力。

> **本项目的代码由 AI 生成。** Android 壳、前端改动与本文档，均由 AI 编码 agent 在人工指导下完成；人类负责提出需求、验收与发布。代码可以正常构建和运行，但请自行评估后再决定是否使用。
>
> **非官方项目。** 与深度求索（DeepSeek）没有任何关系，未获其授权或背书。见文末[免责声明](#免责声明)。

---

## 这是什么

官方 chat-demo 是一个纯静态页面：填自己的 API Key，浏览器直连 `api.deepseek.com`。它在桌面浏览器上很好用，但装到手机上体验是断的——没有图标、带地址栏、切后台就丢状态、点图片没反应。

这个项目做了两件事：

1. **一个极简 WebView 壳**（`android/`），把前端打进 APK，提供图标、全屏、返回键、相机、麦克风、文件桥接；
2. **对前端做了一批改造**（`frontend/`），补上官方 demo 没做的功能。

整个壳只有一个第三方依赖：`androidx.webkit`（用来把本地静态文件挂到 https 源上，否则 IndexedDB 不可靠）。

## 功能

**在官方 chat-demo 基础上新增**

| 功能 | 说明 |
|---|---|
| 角色设定 | 每个会话单独设置，新建会话时预填默认值。服务端会用自己的系统提示词覆盖 `system` 字段，所以这里额外用一段前置问答把设定注入上下文，才真正生效 |
| 每轮用量 | 回复下方显示 `输入 · 输出 · 上下文%`，单行、窄屏省略号收敛 |
| 文件管理 | 列出账号下已上传的文件（`GET /anthropic/v1/files`），可逐个删除 |
| 本地附件管理 | 列出手机上保存的附件原件、被哪些会话引用，可删除或清理未引用项 |
| 拍照提问 | 调起系统相机（完整相机功能），拍完压缩后直接进附件 |
| 图片查看 | 点缩略图全屏查看，支持双指缩放、拖动、双击 |
| 导入/导出进度条 | 长任务有可见进度，总数未知时先跑无限动画 |
| 推理强度 | 设置里可选低/标准/最高，走 Anthropic 格式的 `output_config.effort` |
| 动画 | 消息、弹窗、菜单、抽屉的过渡，并尊重系统「减少动态效果」 |
| 清除缓存 | 清掉导出/导入的暂存文件和 WebView 缓存 |

**壳提供的能力**

- 边到边显示，状态栏/导航栏透明，页面背景铺满
- 返回键分层处理：收键盘 → 关弹窗 → 收抽屉 → 回新对话 → 双击退出
- 麦克风权限桥接（WebView 需要单独实现 `onPermissionRequest`）
- 大文件导入时先把 zip 复制到私有目录，绕开 Android SAF 的随机读性能问题
- 失败兜底：麦克风约束降级重试、录音采样率降级重采样

## 仓库结构

```
shenliao/
├─ android/                  Android 壳（Kotlin，Gradle）
│  ├─ app/src/main/
│  │  ├─ java/.../MainActivity.kt
│  │  ├─ assets/www/         构建时放前端产物（当前是占位页）
│  │  └─ res/
│  ├─ prepare-assets.ps1     把前端 dist 拷进 assets
│  └─ build-apk.ps1          找 JDK/SDK/Gradle 并构建
├─ frontend/                 改造过的 chat-demo 源码（TypeScript + React）
│  └─ src/
├─ LICENSE                   MIT（来自上游）
└─ NOTICE.md                 署名与改动说明
```

## 构建

**环境要求**：Node.js 22.12+、pnpm、JDK 17+、Android SDK（platforms;android-35、build-tools;35.0.0）。

```powershell
# 1. 构建前端
cd frontend
pnpm install
pnpm build            # 产物在 frontend/dist

# 2. 塞进壳
cd ..\android
.\prepare-assets.ps1 -Dist ..\frontend\dist

# 3. 构建 APK
.\build-apk.ps1
# 产物：android\app\build\outputs\apk\debug\app-debug.apk
```

`build-apk.ps1` 会自动找 JDK（`JAVA_HOME` → Android Studio 自带的 JBR）、Android SDK、Gradle；也可以用 `-Jdk` / `-Sdk` / `-Gradle` 显式指定。

用 Android Studio 打开 `android/` 目录同样可以构建。

### 两种运行模式

| | 内置（默认） | 远程 |
|---|---|---|
| 页面来源 | APK 内的 `assets/www` | 构建时 `-PchatDemoUrl=<url>` 指定的线上地址 |
| 断网 | 界面能开 | 白屏 |
| 更新 | 需重新打包 | 自动跟随 |

```powershell
.\build-apk.ps1 -Remote https://cdn.deepseek.com/chat-demo/
```

## 使用

1. 安装 APK，首次启动进「设置」填自己的 DeepSeek API Key
2. 开始对话

数据都存在本机：会话和附件原件在 IndexedDB，设置和 Key 在 localStorage。**没有任何服务器**，除了 `api.deepseek.com` 之外不发请求。

## 已知限制

- **不是官方应用**，接口随时可能变化
- 附件原件会同时存在本机和 DeepSeek 服务器上；服务器那份**不能下载**，所以删掉本地副本就再也看不到那张图了
- 自定义的角色设定不会进「导出历史」的 ZIP（归档格式是官方 v3，没扩展）
- 上下文窗口按 1M 估算（`frontend/src/utils/format.ts` 里的 `CONTEXT_WINDOW`），换模型需要自己改
- 只在 Android 上验证过，`minSdk 29` / `targetSdk 34`

## 免责声明

本项目是个人学习性质的第三方客户端，**与深度求索（DeepSeek）无关**，未获其授权、认可或支持。项目不包含任何官方商标素材，图标为自制矢量图形。

使用本项目需要你自己的 DeepSeek API Key，产生的费用由你自行承担。请遵守 [DeepSeek 服务条款](https://cdn.deepseek.com/policies/zh-CN/deepseek-terms-of-use.html)。

软件按「现状」提供，不附带任何担保。代码由 AI 生成，可能存在尚未被发现的问题；请在理解其行为的前提下使用，风险自负。

## 许可

MIT，见 [LICENSE](LICENSE)。前端部分来自 [deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo)（MIT），署名与改动清单见 [NOTICE.md](NOTICE.md)。
