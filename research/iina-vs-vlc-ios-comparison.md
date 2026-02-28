# iina/iina vs videolan/vlc-ios 对比

更新时间：2026-02-27

## 1. 对比前提

这两个项目都叫“播放器”，但定位不同：

- `iina/iina`：macOS 桌面播放器产品
- `videolan/vlc-ios`：iOS/tvOS/visionOS/watchOS 播放器产品

因此它们不是同平台同约束下的 1:1 竞争，而是“同类问题在不同平台的架构实现”。

## 2. 一句话结论

- IINA 路线：**Swift/AppKit 应用层 + libmpv + FFmpeg（手工 dylib 集成）**
- VLC-iOS 路线：**ObjC/Swift 应用层 + VLCKit(libvlc wrapper) + CocoaPods 依赖体系**

## 3. 核心技术栈对比

| 维度 | IINA | VLC-iOS |
|---|---|---|
| 平台 | macOS | iOS / tvOS / visionOS / watchOS |
| 播放内核 | libmpv（底层依赖 FFmpeg） | libvlc（通过 VLCKit 封装） |
| 主要语言 | Swift 为主，少量 ObjC/C 桥接 | Objective-C + Swift 混合 |
| 依赖管理 | SPM + 预编译/自编译 `.dylib` | CocoaPods（VLCKit、VLCMediaLibraryKit 等） |
| UI 技术 | AppKit + XIB（少量 SwiftUI） | UIKit/TVUIKit + XIB + Swift/ObjC 视图控制器 |
| 扩展体系 | 内建 JavaScript 插件系统（JavaScriptCore/WebKit） | 未见类似脚本插件运行时（更偏原生功能堆叠） |

## 4. 关键证据（代码/配置）

### 4.1 IINA

- README 明确基于 mpv：`README.md`（Based on mpv / uses mpv for media playback）
- libmpv/FFmpeg 桥接：`iina/iina-Bridging-Header.h`
- 多 target（App + Extension + CLI + plugin tool）：`iina.xcodeproj/project.pbxproj`
- SPM 依赖：PromiseKit / Mustache / Sparkle / Just：`iina.xcodeproj/project.pbxproj`
- 插件系统：`iina/JavascriptPlugin.swift`、`iina/JavascriptAPI*.swift`、`iina-plugin/main.swift`

### 4.2 VLC-iOS

- README 明确使用 VLCKit（libvlc wrapper）：`vlc-ios-upstream/README.md`
- CocoaPods 核心依赖：`VLCKit`、`VLCMediaLibraryKit`：`vlc-ios-upstream/Podfile`
- 桥接头直接引入 VLCKit：`vlc-ios-upstream/Sources/Headers/VLC-iOS-Bridging-Header.h`
- 多平台 target 与测试 target：`vlc-ios-upstream/VLC.xcodeproj/project.pbxproj`
- GitLab CI + SwiftLint：`vlc-ios-upstream/Buildsystem/gitlab-ci.yml`、`Buildsystem/swiftlint.yml`

## 5. 架构风格差异

### 5.1 内核接入方式

- IINA：直接面向 `libmpv` 的命令/属性/事件模型编排（`MPVController`）
- VLC-iOS：通过 `VLCKit` 暴露的 `VLCMedia/VLCMediaList/VLCMediaPlayer` 抽象来调用

直观理解：

- IINA 更像“深入播放器内核接口层”
- VLC-iOS 更像“使用 libvlc 的上层 SDK 抽象层”

### 5.2 工程依赖哲学

- IINA：媒体内核二进制由项目脚本管理（`other/download_libs.sh` + `deps/lib`），对库版本和打包链路控制更直接
- VLC-iOS：以 CocoaPods 统一依赖，VLCKit 版本在 Podfile 中固定，生态依赖更集中

### 5.3 产品功能重心

- IINA：桌面播放器体验 + 插件脚本扩展 + CLI/浏览器联动
- VLC-iOS：移动端多平台适配 + 网络服务浏览/云盘集成 + CarPlay/远程控制等场景功能

## 6. 对“想理解 IINA”的学习价值

如果你的目标是理解 IINA，VLC-iOS 最有价值的是“对照视角”，不是主线源码：

1. 看它如何组织复杂播放器产品（多 target、多平台、多服务接入）。
2. 看它如何通过 VLCKit 抽象内核能力。
3. 再回到 IINA，对比为什么 IINA 选择 `libmpv` 直连路线与插件化策略。

建议主次：

- 主线：IINA -> mpv -> FFmpeg
- 对照：VLC-iOS（验证你的架构判断）

## 7. 你可以直接做的对照练习

1. 选同一需求（如“打开网络流并播放”），分别在 IINA 与 VLC-iOS 追调用链。
2. 各写一页说明：调用入口、状态变化、错误处理、可扩展点。
3. 输出最终结论：哪一条路线更适合你未来的 iOS 播放器目标（成本/能力/维护）。

## 8. FAQ：VLCKit / libVLC 是不是 `mpv + FFmpeg`？

结论分两句：

1. **libVLC 是开源的。**  
2. **libVLC/VLCKit 路线不是 `mpv + FFmpeg`。**

具体解释：

- `VLCKit` 是 `libVLC` 的 Apple 平台封装（wrapper），官方 README 明确写了 “Wrapper of libVLC”。  
- `libVLC` 是 VLC 的可嵌入引擎，VLC 官方 README 明确写了 `libVLC` 采用 `LGPLv2(or later)`。  
- `libvlc.h` 文件头也写明 LGPL 2.1+ 授权。  
- 在技术实现上，VLC/libVLC 是自己的模块化播放器架构；它**可以**使用 FFmpeg/libavcodec 模块（例如 `modules/codec/avcodec/video.c`），但这不等于 mpv 路线。  

一句话区分：

- IINA：`libmpv + FFmpeg`
- VLC-iOS：`VLCKit(libVLC) + VLC 模块体系（其中可包含 FFmpeg/libavcodec 模块）`

## 9. 参考来源

- VLCKit README（Wrapper of libVLC）：<https://code.videolan.org/videolan/VLCKit/-/raw/master/README.md>
- VLCKit 构建脚本（编译 libvlc）：<https://code.videolan.org/videolan/VLCKit/-/raw/master/compileAndBuildVLCKit.sh>
- VLC-iOS README（uses VLCKit）：<https://github.com/videolan/vlc-ios/blob/master/README.md>
- VLC README（libVLC 为 LGPLv2+）：<https://raw.githubusercontent.com/videolan/vlc/master/README.md>
- libVLC API 头文件（LGPLv2.1+）：<https://raw.githubusercontent.com/videolan/vlc/master/include/vlc/libvlc.h>
- VLC avcodec 模块示例（使用 libavcodec）：<https://raw.githubusercontent.com/videolan/vlc/master/modules/codec/avcodec/video.c>
