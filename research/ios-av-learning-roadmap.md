# iOS 音视频入门学习路线（含开源项目实战）

更新时间：2026-02-26

## 结论先说

`ijkplayer` 适合做“播放器内核与 FFmpeg 机制”的进阶学习，不建议作为 iOS 音视频入门第一站。入门建议先走 **AVFoundation 原生链路**，再看 **现代 Swift 开源项目（如 HaishinKit）**，最后回看 `ijkplayer` 做底层补课。

## 为什么不建议先从 ijkplayer 入门

- 历史包袱较重，工程与依赖链更偏“维护型学习”。
- 对初学者来说，FFmpeg + 播放器内核 + iOS UI 同时学习，认知负担太高。
- 你会更容易“会改配置”，但不容易先建立 iOS 音视频主干认知（采集、编码、播放、渲染、时钟同步、缓冲策略）。

## 推荐学习顺序（4 周）

### 第 1 周：原生基础（必须）

目标：建立 iOS 音视频主干模型。

- 学习 `AVFoundation` 关键模块：
  - 采集：`AVCaptureSession` / `AVCaptureDevice`
  - 播放：`AVPlayer` / `AVPlayerItem`
  - 导出：`AVAsset` / `AVAssetExportSession`
- 跑通 Apple 官方示例（相机与播放链路）。
- 输出：画一张你自己的“采集 -> 编码 -> 传输 -> 解码 -> 渲染”流程图。

推荐资料：

- https://developer.apple.com/documentation/avfoundation
- https://developer.apple.com/documentation/http-live-streaming

### 第 2 周：对着现代 Swift 开源项目读代码（推荐 HaishinKit）

目标：理解直播推流实战（RTMP/SRT）在 iOS 的工程组织。

- 项目：`HaishinKit.swift`
- 重点看：
  - Camera/Microphone Capture 管线
  - RTMP/SRT 连接与重连
  - 编码参数（码率/GOP/关键帧）
  - 后台、权限、网络抖动处理
- 输出：做一个最小 Demo（前置摄像头 + 麦克风推流到测试地址）。

仓库：

- https://github.com/HaishinKit/HaishinKit.swift

### 第 3 周：播放器链路与缓存策略

目标：搞懂“播起来”和“播得稳”是两回事。

- 学习播放侧核心问题：
  - 首帧耗时
  - 卡顿恢复
  - 音画同步（A/V Sync）
  - 缓冲区策略
- 可选对照项目：
  - `mobileplayer-ios`（播放器工程结构）
  - `BMPlayer`（AVPlayer 封装思路）

### 第 4 周：底层补课（再看 ijkplayer）

目标：理解 FFmpeg 型播放器内核设计。

- 项目：`ijkplayer`
- 重点看：
  - FFmpeg 集成方式
  - 解复用与解码线程模型
  - 视频渲染与音频输出路径
  - 上层控制接口与状态机
- 输出：写一篇“AVPlayer vs ijkplayer”对比（可维护性、延迟、兼容性、可控性）。

仓库：

- https://github.com/bilibili/ijkplayer

## 每天学习模板（90~120 分钟）

- 20 分钟：读文档/源码（只抓 1 个问题）
- 40 分钟：跑代码并加日志
- 20 分钟：写实验记录（输入/操作/现象/结论）
- 10~40 分钟：做一个微改动并验证

## 开源项目学习法（避免“只看不练”）

- 只盯一条链路：例如“点击开始推流后发生了什么”。
- 强制产出：每天至少 1 个可运行改动。
- 先观测再改动：先加日志，再动逻辑。
- 先稳定再优化：先跑通，再调延迟/画质/功耗。

## 里程碑验收标准

- 能独立解释 iOS 采集、编码、推流、播放 4 条主链路。
- 能做出一个最小 RTMP/SRT 推流 Demo。
- 能定位一次真实卡顿问题（网络、编码、渲染任一层）。
- 能说清楚为什么 `ijkplayer` 适合进阶而不是入门第一站。

## 推荐仓库（按入门友好度）

1. HaishinKit.swift（现代 Swift + 推流实战）
2. mobileplayer-ios / BMPlayer（播放器工程组织）
3. ijkplayer（底层进阶与 FFmpeg 机制）

