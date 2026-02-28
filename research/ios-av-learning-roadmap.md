# iOS 音视频入门学习路线（含开源项目实战）

更新时间：2026-02-27

## 结论先说

`ijkplayer` 适合做“播放器内核与 FFmpeg 机制”的进阶学习，不建议作为 iOS 音视频入门第一站。  
入门更合理的顺序是：**采集 -> 播放 -> 录制 -> 推流 -> 播放器内核**。  
也就是先打牢 `AVFoundation/AVPlayer`，再上 `HaishinKit` 推流，最后看 `IINA/ijkplayer` 做内核与工程化补课。

## 为什么不建议先从 ijkplayer 入门

- 历史包袱较重，工程与依赖链更偏“维护型学习”。
- 对初学者来说，FFmpeg + 播放器内核 + iOS UI 同时学习，认知负担太高。
- 你会更容易“会改配置”，但不容易先建立 iOS 音视频主干认知（采集、编码、播放、渲染、时钟同步、缓冲策略）。

## 为什么不要一上来就推流

- 推流同时叠加了：采集、编码、网络、协议、重连、弱网稳定性。
- 新手在推流阶段遇到问题时，常常分不清是“采集问题”还是“网络问题”。
- 先把采集/播放/录制打通，再做推流，排障路径会清晰很多。

## 推荐学习顺序（6 周）

### 第 1 周：原生基础（必须）

目标：建立采集与会话管理主干能力。

- 学习 `AVFoundation` 关键模块：
  - 采集：`AVCaptureSession` / `AVCaptureDevice`
  - 音频会话：`AVAudioSession`
- 跑通相机预览 + 麦克风采集 + 权限申请。
- 输出：最小 Demo（预览、切前后摄、静音开关）。

推荐资料：

- https://developer.apple.com/documentation/avfoundation
- https://developer.apple.com/documentation/http-live-streaming

### 第 2 周：先学播放链路（AVPlayer）

目标：理解播放器最基础的时序与缓冲行为。

- 学习：
  - `AVPlayer` / `AVPlayerItem`
  - 首帧耗时、缓冲状态、卡顿恢复
  - 音画同步（A/V Sync）基本概念
- 输出：最小播放器 Demo（本地文件 + HLS URL）。

### 第 3 周：录制与媒体处理（本地闭环）

目标：把“采集 -> 编码 -> 文件输出”跑通。

- 学习：
  - `AVAssetWriter` / `AVAsset` / `AVAssetExportSession`
  - 码率/分辨率/帧率/GOP 对文件与画质影响
- 输出：录制 Demo（录像、保存相册、导出不同清晰度）。

### 第 4 周：推流实战（HaishinKit）

目标：在已有采集基础上增加 RTMP/SRT 推流。

- 项目：`HaishinKit.swift`
- 重点看：
  - Camera/Microphone Capture 到网络发送的完整链路
  - RTMP 与 SRT 的连接与重连
  - 编码参数（码率/GOP/关键帧）动态调整
  - 弱网场景下的降档与恢复策略
- 输出：最小推流 Demo（开始/停止、重连、码率切换）。

仓库：

- https://github.com/HaishinKit/HaishinKit.swift

### 第 5 周：工程化补课（加入 IINA）

目标：理解“桌面级播放器产品”如何组织 `libmpv + FFmpeg` 能力。

- 项目：`IINA`
- 重点看：
  - `MPVController`：应用层如何驱动 libmpv
  - Objective-C Bridging：Swift 与 C/FFmpeg 的桥接方式
  - 多 target 工程组织（App / CLI / Extension）
  - 插件系统（JavaScriptCore + WebKit 消息桥）
- 输出：写一篇“HaishinKit/AVPlayer 项目 vs IINA 架构”对比（可扩展性、维护成本、能力边界）。

仓库：

- https://github.com/iina/iina

### 第 6 周：底层补课（再看 ijkplayer）

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

- 能独立解释 iOS 采集、编码、播放、录制、推流 5 条主链路。
- 能做出一个最小播放器 Demo（本地 + HLS）。
- 能做出一个最小录制 Demo（录制 + 导出）。
- 能做出一个最小 RTMP/SRT 推流 Demo。
- 能定位一次真实卡顿问题（网络、编码、渲染任一层）。
- 能解释 IINA 这类“产品级播放器”如何在应用层封装 `libmpv + FFmpeg`。
- 能说清楚为什么 `ijkplayer` 适合进阶而不是入门第一站。

## 推荐仓库（按入门友好度）

1. Apple AVFoundation/AVPlayer 官方文档与示例（采集/播放基础）
2. HaishinKit.swift（推流实战：RTMP/SRT）
3. mobileplayer-ios / BMPlayer（播放器工程组织）
4. IINA（`libmpv + FFmpeg` 的产品工程实践）
5. ijkplayer（底层进阶与 ffplay/FFmpeg 机制）
