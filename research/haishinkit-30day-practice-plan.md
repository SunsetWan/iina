# HaishinKit 30 天实战项目清单（iOS 音视频）

更新时间：2026-02-26

## 目标

30 天内做出一个可用的 iOS 推流 Demo，具备：

- 摄像头 + 麦克风采集
- RTMP/SRT 推流
- 弱网重连与基础自适应
- 本地录制与实时状态面板

## 使用仓库

- https://github.com/HaishinKit/HaishinKit.swift

## HaishinKit 背后技术栈

### 1) 语言与并发模型

- 主要语言：Swift（仓库语言统计以 Swift 为主）
- Swift 工具链：`swift-tools-version: 6.0`
- 并发特性：支持 Strict Concurrency（README 明确）

### 2) 平台与系统能力

- 支持平台：iOS / tvOS / macOS / Mac Catalyst / visionOS
- iOS 侧核心系统框架：`AVFoundation`（音视频采集与会话）
- 屏幕采集能力：ReplayKit（iOS）、ScreenCaptureKit（macOS）

### 3) 传输协议与模块化设计

- 传输协议：RTMP、SRT，并提供 WHEP/WHIP（alpha）
- SPM 产品拆分：
  - `HaishinKit`（基础能力）
  - `RTMPHaishinKit`
  - `SRTHaishinKit`
  - `MoQTHaishinKit`
  - `RTCHaishinKit`

### 3.1) 什么是 RTMP/SRT 推流

- 推流（Publish）：
  - 指 App 把“摄像头/麦克风采集并编码后的音视频数据”持续发送到直播服务器。
  - 服务器再把流分发给观众端（播放端）。

- RTMP 推流（Real-Time Messaging Protocol）：
  - 直播领域最常见的传统推流协议之一，生态成熟，接入成本低。
  - 通常用于“主播端 -> CDN/直播平台入口”。
  - 优点：平台兼容和历史支持好；缺点：在复杂弱网下稳定性与抗抖动能力相对一般。

- SRT 推流（Secure Reliable Transport）：
  - 面向公网复杂网络的实时传输协议，强调抗丢包、抗抖动与加密传输。
  - 优点：弱网稳定性通常更好，延迟控制更灵活；缺点：接入端和运维链路复杂度通常高于 RTMP。

- 实战选型建议：
  - 平台要求 RTMP 或你要快速接入：优先 RTMP。
  - 网络条件复杂、对稳定性要求更高：优先 SRT。
  - 工程上常见做法：保留 RTMP 与 SRT 双通道能力，根据业务场景切换。

### 4) 关键依赖

- `Logboard`（日志）
- `libsrt`（SRT 二进制 XCFramework）
- `libdatachannel`（RTC 相关二进制 XCFramework）

### 5) 版本与要求（当前主线）

- 开发环境：Xcode 16.4+/26.0+，Swift 6.0+
- OS 基线：iOS 15+（以及对应 tvOS/macOS/visionOS 基线）
- 许可证：BSD-3-Clause

## 第 1 周：跑通最小链路

Day 1：搭建工程，集成 HaishinKit，跑通编译。  
Day 2：申请相机/麦克风权限，显示本地预览。  
Day 3：接入最小 RTMP 推流（开始/停止）。  
Day 4：接入最小 SRT 推流（开始/停止）。  
Day 5：加前后摄切换、静音开关。  
Day 6：整理推流状态机（Idle/Connecting/Streaming/Failed）。  
Day 7：验收 1：录屏演示“可推可停可切换”。

## 第 2 周：编码与画质控制

Day 8：增加分辨率切换（360p/720p/1080p）。  
Day 9：增加码率切换（低/中/高）。  
Day 10：增加帧率切换（15/24/30）。  
Day 11：加入关键帧间隔（GOP）配置。  
Day 12：抽象 `StreamProfile`（一组编码参数模板）。  
Day 13：实现“推流中动态切档”。  
Day 14：验收 2：不同配置下画质/延迟对比记录。

## 第 3 周：稳定性（弱网与重连）

Day 15：增加连接事件日志（连接、断开、错误码）。  
Day 16：实现指数退避重连（1s/2s/4s/... 上限）。  
Day 17：加网络切换处理（Wi-Fi <-> 蜂窝）。  
Day 18：加超时保护（连接超时、发布超时）。  
Day 19：实现自动降档策略（先降码率，再降分辨率）。  
Day 20：实现恢复策略（网络恢复后逐步升档）。  
Day 21：验收 3：弱网模拟下连续推流 20 分钟。

## 第 4 周：可用性与可观测性

Day 22：实现本地录制（边推边录）。  
Day 23：录制文件管理（命名、列表、删除）。  
Day 24：增加状态面板（FPS/码率/重连次数/当前档位）。  
Day 25：导出调试日志（文本文件）。  
Day 26：加入温控与后台策略（高负载降档）。  
Day 27：处理中断（来电/音频会话变化）恢复。  
Day 28：验收 4：完整流程演示（推流+录制+重连+面板）。

## 收尾 2 天：文档与发布前检查

Day 29：补 README（架构、配置说明、已知问题）。  
Day 30：做回归清单与发布包（TestFlight 或内测包）。

## 每日交付模板

- 今日任务：  
- 关键改动文件：  
- 运行结果：  
- 遇到问题：  
- 明日计划：  

## 最终验收标准

- 能稳定推 RTMP 和 SRT（至少各 15 分钟）  
- 弱网下能自动重连并尽量不断流  
- 支持动态切档（码率/分辨率）  
- 支持边推边录和日志导出  
- 有一份可复现实验记录（延迟、卡顿、重连统计）
