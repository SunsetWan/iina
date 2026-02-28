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

---

## 11. mpv / FFmpeg 是否开源？各自技术栈是什么

> 注：你问题里写的 `ffmpg` 一般是 `FFmpeg`。

### 11.1 mpv

- 是否开源：**是**
- 许可证：默认 **GPLv2 or later**；可用 `-Dgpl=false` 以 **LGPLv2.1 or later** 方式构建
- 主要技术栈：
  - 主要语言：C（GitHub 语言占比约 87.8%），并含 Lua、Swift、Python 等辅助部分
  - 构建系统：Meson（`meson.build` / `meson.options`）
  - 关键依赖：FFmpeg（`libavcodec/libavformat/libavutil/libswscale/libavfilter`）、libplacebo、libass、Lua 等

### 11.2 FFmpeg

- 是否开源：**是**
- 许可证：代码库主体是 **LGPL**，可选组件为 **GPL**（官方 README 明确）
- 主要技术栈：
  - 主要语言：C（GitHub 语言占比约 89.8%），并有 Assembly、Makefile、少量 C++ 等
  - 构建系统：`configure` + `Makefile`
  - 核心库模块：`libavcodec`、`libavformat`、`libavutil`、`libavfilter`、`libavdevice`、`libswresample`、`libswscale`

### 11.3 关系简述

- FFmpeg 是底层多媒体库集合（编解码、封装、滤镜、设备与工具链）
- mpv 是播放器内核，构建时依赖 FFmpeg 提供解码/封装等底层能力

### 11.4 外部证据来源（官方）

- mpv 仓库：<https://github.com/mpv-player/mpv>
- FFmpeg 仓库：<https://github.com/FFmpeg/FFmpeg>

---

## 12. 为什么这类播放器不直接使用 AVPlayer

### 12.1 AVPlayer 适合的场景

- 面向苹果生态标准播放链路（尤其 HLS）
- 系统集成能力强（AirPlay、PiP、FairPlay DRM、系统级功耗优化）
- 开发与维护成本较低，稳定性收益高

### 12.2 选择 mpv/FFmpeg 的核心动机

- 需要更广的媒体格式与编解码覆盖能力
- 需要更高的可定制性：滤镜链、渲染参数、字幕能力、播放行为控制
- 需要脚本化/插件化扩展生态（如 Lua/JavaScript）
- 产品定位偏“专业播放器内核能力”，而非“标准业务播放组件”

### 12.3 代价与权衡

- 工程复杂度和维护成本显著上升
- 二进制依赖管理更复杂（版本、架构、签名、发布链路）
- 系统升级或底层库升级时，兼容性与回归成本更高

### 12.4 选型经验

- 目标是标准业务播放（主流格式、苹果生态优先）：优先 `AVPlayer`
- 目标是高兼容度/高可控度/专业播放器能力：优先 `mpv + FFmpeg`

---

## 13. IINA vs bilibili ijkplayer：核心技术对比

### 13.1 核心结论

- IINA：播放器内核是 **`libmpv + FFmpeg`**
- ijkplayer：播放器内核是 **`ffplay/ijk 播放层 + FFmpeg`**（不是 mpv）

也就是说，二者都依赖 FFmpeg，但“播放器内核层”不同：

- IINA 借助 `libmpv` 作为完整播放器内核
- ijkplayer 走 `ffplay` 路线并演进为自己的播放器层（iOS/Android 双端封装）

### 13.2 ijkplayer 技术栈（重点）

- 开源：是（公开仓库 `bilibili/ijkplayer`）
- 许可证：仓库元数据为 `GPL-2.0`；README 说明主体按 `LGPLv2.1 or later` 分发并包含多方依赖许可
- 主要语言（GitHub 统计）：C、Objective-C、Java、Shell、Makefile
- 核心媒体能力：FFmpeg（README 明确 “Video player based on ffplay”）
- iOS 侧典型栈：
  - 视频输出：OpenGL ES 2.0
  - 音频输出：AudioQueue、AudioUnit
  - 硬解：VideoToolbox（iOS 8+）
  - 对外接口：MediaPlayer-like 的 Objective-C API 封装
- 构建链路：Shell 脚本 + FFmpeg 编译脚本 + Xcode 工程集成（`init-ios.sh`、`compile-ffmpeg.sh`）

### 13.3 与 IINA 的工程差异（架构视角）

- IINA（macOS 桌面产品）：
  - 业务层：Swift/AppKit
  - 播放内核：libmpv
  - 解码/封装：FFmpeg 动态库
- ijkplayer（跨端播放器 SDK）：
  - 业务层：iOS Objective-C 封装 + Android Java 封装
  - 播放内核：ffplay 路线的 ijk 播放层
  - 解码/封装：FFmpeg

### 13.4 一句话回答你的问题

如果 IINA 的核心是 `mpv + FFmpeg`，那么 ijkplayer 的核心更接近 **`ffplay/ijk 内核 + FFmpeg`**。

---

## 14. ffplay 与 ffplay/ijk 内核：介绍与分析

### 14.1 ffplay 是什么

- `ffplay` 是 FFmpeg 官方仓库中的参考级播放器程序（`fftools/ffplay.c`）
- 本质是：在 `libavformat/libavcodec/libavfilter/libswresample/libswscale` 之上，提供最小可用的播放调度与渲染逻辑
- 渲染与事件循环主要依赖 SDL（`SDL`、`SDL_thread`）

从源码结构看，`ffplay` 的核心模型通常包含：

- `read thread`：读包与分发
- `packet queue`：音视频包队列
- `decoder thread`：音频/视频/字幕解码线程
- `frame queue`：解码帧队列
- `clock`：音频时钟 / 视频时钟 / 外部时钟三时钟同步策略

### 14.2 ffplay 的定位与优缺点

- 定位：教学/验证/参考实现（不是给 App 直接嵌入的稳定 SDK API）
- 优点：
  - 结构清晰，便于理解 FFmpeg 播放主链路
  - 对同步、队列、解码调度提供“可运行参考实现”
- 局限：
  - UI 与平台集成能力较弱（偏工具程序）
  - 可扩展性与产品化能力有限
  - 直接用于移动端/桌面产品时，通常需要大量改造

### 14.3 什么是 ffplay/ijk 内核

- `ijkplayer` 中存在 `ijkmedia/ijkplayer/ff_ffplay.c`，其头部说明了 ffplay 基底
- 可理解为：以 ffplay 思路为基础，向移动端产品化演进出的 “ijk 播放内核层”
- 在 ijk 中，这层并非单文件，常配套：
  - `ff_ffpipeline.*` / `ff_ffpipenode.*`：播放管线抽象
  - `ijkplayer.*`：对外播放器控制接口
  - `ijksdl` 相关封装：跨平台线程/同步/渲染适配

### 14.4 ffplay/ijk 相比原生 ffplay 的工程化改造方向

- 更强的跨平台适配（iOS/Android）
- 更完整的播放器状态管理与消息系统（更偏 SDK 化）
- 更贴近业务需求的缓冲、重试、统计、硬解开关等策略
- 为上层提供 Objective-C/Java API，而不是仅命令行入口

### 14.5 与 IINA（libmpv 路线）的本质差异

- ffplay/ijk 路线：团队维护“播放器内核层”本身（自由度高，但维护成本更高）
- libmpv 路线：复用成熟播放器内核，再聚焦业务层与产品层（工程风险相对可控）

一句话：

- `ffplay/ijk` 更像“自己养内核”
- `libmpv` 更像“复用内核做产品”
