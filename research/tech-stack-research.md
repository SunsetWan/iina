# IINA 项目技术栈深度调研

## 1. 调研范围与方法

本报告基于仓库源码与配置文件静态分析，重点检查了：

- 工程配置：`iina.xcodeproj/project.pbxproj`、`Configs/*.xcconfig`
- 构建与发布：`README.md`、`.github/workflows/*.yml`、`other/*.sh|*.rb`
- 运行时代码：`iina/`、`OpenInIINA/`、`iina-cli/`、`iina-plugin/`、`browser/`
- 本地化与质量工具：`crowdin.yml`、`.typos.toml`、`other/check_*.swift`

---

## 2. 技术栈总览（结论）

IINA 是一个**以 Swift + AppKit（Cocoa）为主**、通过 **Objective-C Bridge** 对接 **libmpv + FFmpeg** 的 macOS 原生播放器项目。  
其依赖管理采用**混合模式**：Swift 包用 SPM，媒体能力由预编译 `.dylib`（或本地构建）注入。  
项目同时包含 Safari App Extension、浏览器扩展（Chrome/Firefox）与两个 CLI 工具（`iina-cli`/`iina-plugin`）。

---

## 3. 语言与 UI/应用层栈

### 3.1 主要语言

- Swift（主语言，`iina` 目录约 179 个 Swift 文件）
- Objective-C（桥接与底层媒体辅助：`FFmpegController.*`、`ObjcUtils.*`、`FixedFontManager.*`）
- JavaScript（Safari 扩展、浏览器扩展）
- Ruby / Shell / Swift Script（构建与代码生成工具链）

### 3.2 UI 技术

- 主体是 **AppKit/Cocoa + XIB**（大量 `*.xib` 与 `import Cocoa`）
- 少量 **SwiftUI**（插件开发工具界面）
- 说明：项目不是以 SwiftUI 为核心，而是传统 macOS 原生 UI 体系

关键证据：

- `iina.xcodeproj/project.pbxproj:2849`（主 target 为 macOS app）
- `iina/JavascriptDevTool.swift:9`（存在 SwiftUI）
- `iina` 目录大量 `.xib` 与本地化 `.strings`

---

## 4. 媒体播放与底层能力栈

### 4.1 核心播放引擎

- **libmpv**（主播放控制核心）
- Swift 层 `MPVController` 直接控制 mpv 事件、属性、渲染参数

关键证据：

- `README.md`（项目明确基于 mpv）
- `iina/MPVController.swift:87`
- `iina.xcodeproj/project.pbxproj:80`（`libmpv.2.dylib in Frameworks`）

### 4.2 编解码与媒体处理

- **FFmpeg 动态库族**（`libavcodec`/`libavformat`/`libavutil`/`libswscale`/`libavfilter` 等）
- FFmpeg 版本主号可见：`libavcodec 61`、`libavformat 61`、`libavutil 59`
- Objective-C 的 `FFmpegController` 负责缩略图、封面等媒体处理能力

关键证据：

- `deps/include/libavcodec/version_major.h:28`
- `deps/include/libavformat/version_major.h:32`
- `deps/include/libavutil/version.h:81`
- `iina/FFmpegController.m:15-20`

### 4.3 图形与系统多媒体框架

- OpenGL（`OpenGL.GL` / `OpenGL.GL3`）
- VideoToolbox（硬解相关能力）
- MediaPlayer、CoreDisplay、私有 PIP.framework 集成

关键证据：

- `iina/MPVController.swift:11`
- `iina.xcodeproj/project.pbxproj:160`（`PIP.framework`）
- `iina.xcodeproj/project.pbxproj:175`（`CoreDisplay.framework`）

---

## 5. 依赖管理与第三方库

### 5.1 Swift Package Manager（SPM）

项目通过 Xcode 的 SPM 引入以下包：

- `PromiseKit`（最低 `6.22.1`）
- `GRMustache.swift`（产品名 `Mustache`，最低 `4.2.0`）
- `Sparkle`（最低 `2.7.0`，自动更新框架）
- `Just`（最低 `0.8.0`，HTTP 客户端）

关键证据：

- `iina.xcodeproj/project.pbxproj:5161-5220`

### 5.2 非 SPM 依赖（媒体库）

- `deps/lib/*.dylib` 由脚本拉取或本地构建后拷贝
- 默认脚本：`other/download_libs.sh` 从 `https://iina.io/dylibs/...` 下载
- 可通过 Homebrew/MacPorts 构建并用 `other/change_lib_dependencies.rb` 重写依赖路径

关键证据：

- `other/download_libs.sh:10-12`
- `other/change_lib_dependencies.rb:57-66`
- `README.md` 的 Building 章节

结论：这是“**SPM + 手工二进制依赖**”的混合依赖体系。

---

## 6. 插件与扩展生态技术栈

### 6.1 IINA 插件系统（应用内）

- 插件运行时：`JavaScriptCore`
- 与 Web UI 交互：`WebKit` + `WKWebView` 消息桥
- 权限模型：网络、文件系统、OSD、Overlay 等权限声明

关键证据：

- `iina/JavascriptPlugin.swift:10-23`
- `iina/JavascriptAPI.swift:10`
- `iina/JavascriptMessageHub.swift:10`

### 6.2 插件开发 CLI（`iina-plugin`）

- Swift 编写，支持 `new/pack/link/unlink`
- 用 Mustache 模板渲染插件模板
- 可生成带 React/Vue/Parcel 选项的插件脚手架

关键证据：

- `iina-plugin/main.swift:10`
- `iina-plugin/main.swift:18-35`
- `iina-plugin/main.swift:129-139`

### 6.3 浏览器与 Safari 扩展

- Safari App Extension：`OpenInIINA`（`SafariServices`）
- Chrome 扩展：Manifest V3（`service_worker`）
- Firefox 扩展：Manifest V2

关键证据：

- `OpenInIINA/SafariExtensionHandler.swift:9`
- `browser/Chrome_Open_In_IINA/manifest.json:2`
- `browser/Firefox_Open_In_IINA/manifest.json:2`

---

## 7. 构建、配置与 CI/CD

### 7.1 工程与构建体系

- Xcode 原生工程（非 Tuist/CMake/CocoaPods/Carthage）
- 多 target：
  - `iina`（主应用）
  - `OpenInIINA`（App Extension）
  - `iina-cli`（命令行工具）
  - `iina-plugin`（插件工具）
- 编译配置通过 `Configs/*.xcconfig` 管理

关键证据：

- `iina.xcodeproj/project.pbxproj:2799-2890`
- `Configs/Shared.xcconfig:57`（Swift 5.0）
- `Configs/Deployment.xcconfig:12-13`（最低系统版本策略）

### 7.2 CI

- GitHub Actions
- `ci.yml`：macOS runner + `xcodebuild` 编译 Nightly 并打包产物
- `spelling.yml`：`crate-ci/typos` 拼写检查

关键证据：

- `.github/workflows/ci.yml`
- `.github/workflows/spelling.yml`

---

## 8. 本地化与工程质量工具链

- Crowdin 同步本地化：`crowdin.yml`
- 语言资源规模大：`iina/*.lproj` 约 55 个
- 本地化校验脚本：`other/check_localizable.swift`、`other/check_translation.swift`
- 代码文本质量：`.editorconfig` + `.typos.toml`

---

## 9. 当前技术特征与风险点

### 9.1 典型技术特征

- 深度依赖本地 macOS 原生 API（AppKit + 私有/系统框架）
- 播放能力强绑定 mpv/FFmpeg 动态库
- 插件生态采用 JS Runtime + WebView Bridge
- 目标形态丰富（GUI + 扩展 + CLI）

### 9.2 可见风险/成本

- 仓库中未见 unit/UI test target 与 XCTest 相关配置，测试主要依赖构建通过与人工验证
- 媒体二进制依赖管理复杂（版本匹配、架构差异、签名与拷贝流程）
- 依赖 OpenGL/私有框架与多架构兼容策略，后续系统升级时需要持续维护

---

## 10. 一句话结论

IINA 的技术栈定位是：**macOS 原生播放器工程（Swift/AppKit）+ mpv/FFmpeg 媒体内核 + JavaScript 插件平台 + Xcode 原生构建体系**，偏“系统级桌面应用”而非通用跨平台应用。
