# 以理解 IINA 为目标的 iOS 音视频入门方案

更新时间：2026-02-27

## 目标澄清

你的目标不是“先做直播推流 App”，而是“理解 IINA 这类播放器是怎么做出来的”。

这类目标最有效的路线是：

1. 先掌握 iOS 端基础音视频主链路（采集/播放/录制）
2. 再拆 IINA 的应用层架构（UI、状态机、播放器封装）
3. 再下潜 mpv/FFmpeg（播放器内核与编解码）

## 推荐开源代码库（按学习优先级）

### A. 核心必读（直接服务 IINA 目标）

1. [iina/iina](https://github.com/iina/iina)
- 为什么看：这是目标样本本体（macOS 应用层 + libmpv 集成范式）。
- 先看这些文件：
  - `iina/MPVController.swift`
  - `iina/PlayerCore.swift`
  - `iina/VideoView.swift`
  - `iina/ViewLayer.swift`
  - `iina/JavascriptPlugin.swift`

2. [mpv-player/mpv](https://github.com/mpv-player/mpv)
- 为什么看：IINA 背后的播放器内核（事件、命令、属性、渲染 API）。
- 先看这些入口：
  - `libmpv/client.h`
  - `DOCS/client-api-changes.rst`
  - `player/`（事件循环与控制核心）

3. [FFmpeg/FFmpeg](https://github.com/FFmpeg/FFmpeg)
- 为什么看：解复用/解码/滤镜的底层能力来源。
- 先看这些入口：
  - `README.md`（库分层）
  - `fftools/ffplay.c`（播放器参考实现）
  - `doc/examples/`（API 使用样例）

4. [mpv-player/mpv-examples](https://github.com/mpv-player/mpv-examples)
- 为什么看：最短路径理解“如何嵌入 libmpv”。
- 用法：先跑最小示例，再映射到 IINA 的 `MPVController`。

5. [haasn/libplacebo](https://github.com/haasn/libplacebo)
- 为什么看：现代视频渲染链路（shader/色彩管理）对 mpv 很关键。
- 说明：放在进阶阶段看，不必第 1 周啃。

### B. 对照阅读（帮助你形成架构判断）

6. [videolan/vlc-ios](https://github.com/videolan/vlc-ios)
- 对照点：另一套成熟播放器产品工程（与 IINA 的架构取舍对比）。

7. [bilibili/ijkplayer](https://github.com/bilibili/ijkplayer)
- 对照点：`ffplay/ijk` 路线，适合理解“自己维护播放器内核层”的成本。

### C. 可选（如果你还想补推流）

8. [HaishinKit/HaishinKit.swift](https://github.com/HaishinKit/HaishinKit.swift)
- 对照点：iOS 推流工程化（RTMP/SRT），不是理解 IINA 的主线，但能补齐实时传输视角。

## 仓库活跃度快照（用于选型，GitHub API）

| 仓库 | Stars | 最近推送（UTC） |
|---|---:|---|
| iina/iina | 43,917 | 2026-02-25T23:28:21Z |
| mpv-player/mpv | 34,231 | 2026-02-27T08:51:14Z |
| FFmpeg/FFmpeg | 57,465 | 2026-02-27T13:24:03Z |
| mpv-player/mpv-examples | 272 | 2024-06-07T10:38:39Z |
| haasn/libplacebo | 703 | 2026-02-18T04:44:43Z |
| videolan/vlc-ios | 1,223 | 2026-02-26T17:39:40Z |
| bilibili/ijkplayer | 33,122 | 2024-08-13T00:53:33Z |
| HaishinKit/HaishinKit.swift | 3,022 | 2026-02-16T16:36:21Z |

## 入门方案（6 周，围绕“理解 IINA”）

### 第 1 周：iOS 基础主链路（不碰推流）

- 目标：把采集/播放/录制三条链路跑通。
- 任务：
  - `AVCaptureSession` 跑通相机+麦克风采集
  - `AVPlayer` 播放本地文件和 HLS
  - `AVAssetWriter` 做最小录制
- 输出：一个最小 iOS Demo（预览 + 播放 + 录制）。

### 第 2 周：开始拆 IINA（应用层）

- 目标：看懂 IINA 如何把“播放器能力”封装成产品功能。
- 任务：
  - 画出 IINA 关键模块图（UI 层、PlayerCore、MPVController）
  - 跟踪 1 条完整链路：打开文件 -> 播放 -> 暂停 -> 关闭
- 输出：模块关系图 + 调用时序图。

### 第 3 周：下潜 mpv（播放器内核）

- 目标：看懂 libmpv 的命令/属性/事件模型。
- 任务：
  - 阅读 `libmpv/client.h`
  - 对照 IINA 中 `MPVController` 的调用点
  - 运行一个 mpv embedding 示例
- 输出：`IINA MPVController` 与 `libmpv API` 对照表。

### 第 4 周：下潜 FFmpeg（媒体处理底座）

- 目标：理解 demux/decode/filter 在播放器中的角色。
- 任务：
  - 阅读 FFmpeg README 库分层
  - 读 `ffplay.c` 里 queue/clock/decode thread 结构
  - 回看 IINA 中 FFmpeg 相关桥接代码
- 输出：一张“播放器数据流”图（packet->frame->render）。

### 第 5 周：架构对照学习（IINA vs VLC-iOS vs ijkplayer）

- 目标：形成“为什么 IINA 选 mpv 路线”的技术判断。
- 任务：
  - 对比 3 条路线：AVPlayer 路线、libmpv 路线、ffplay/ijk 路线
  - 记录每条路线的能力边界与维护成本
- 输出：选型对比文档（可维护性/扩展性/兼容性/复杂度）。

### 第 6 周：做一个“Mini-IINA-iOS”练习项目

- 目标：把学到的架构思想落地，而不是复制 IINA 全功能。
- 建议功能：
  - 播放列表
  - 字幕开关
  - 播放控制状态机
  - 调试面板（首帧耗时/缓冲状态）
- 输出：
  - 可运行 Demo
  - 架构说明（模块边界 + 后续如何替换内核）

## 一句话建议

如果你的目标是理解 IINA：先把 iOS 音视频基础链路打通，再“以 IINA 为主、mpv/FFmpeg 为核”去拆架构；推流（HaishinKit）是可选补充，不是主线。
