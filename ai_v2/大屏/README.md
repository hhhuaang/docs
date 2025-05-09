# 大屏接口文档集

本目录包含了所有与大屏端相关的 Socket.io 接口文档，用于指导开发人员实现大屏端与各测试设备之间的通信和数据交互。

## 文档概述

这些文档详细描述了体育测试系统中各个模块与大屏端的通信接口，基于 Socket.io 技术实现实时数据传输和状态同步。所有接口均采用统一的消息格式，提供了标准化的通信规范。

## 文档列表

| 文档名称 | 描述 |
|---------|------|
| [消息格式规范.md](消息格式规范.md) | Socket.io 通用消息格式规范，定义了所有模块使用的标准消息结构 |
| [仰卧起坐.md](仰卧起坐.md) | 仰卧起坐测试系统与大屏端的通信接口 |
| [立定跳远.md](立定跳远.md) | 立定跳远测试系统与大屏端的通信接口 |
| [肢体远程控制.md](肢体远程控制.md) | 肢体远程控制系统与大屏端的通信接口 |
| [摸高.md](摸高.md) | 摸高测试系统与大屏端的通信接口 |
| [开合跳.md](开合跳.md) | 开合跳测试系统与大屏端的通信接口 |
| [引体向上.md](引体向上.md) | 引体向上测试系统与大屏端的通信接口 |
| [遥控接收.md](遥控接收.md) | 大屏接收遥控器控制命令的接口文档 |
| [人脸识别.md](人脸识别.md) | 人脸识别系统与大屏端的通信接口 |

## 通用连接方式

所有测试模块使用相同的连接地址和方式：

- **URL**: `wss://gs.classtao.cn/all`
- **传输协议**: WebSocket
- **连接参数**:
  - `roomId`: 大屏的硬件id，唯一不变
  - `type`: 客户端类型，大屏端应设置为 `"display"`

**连接示例**:
```javascript
const socket = io("wss://gs.classtao.cn/all", {
  transports: ["websocket"],
  query: {
    roomId: "5e110d3a9f236c7de393856bb894b6b7",
    type: "display"
  }
});
```

## 消息处理流程

大屏端处理各种测试模块消息的通用流程如下：

1. **连接服务器**: 建立 Socket.io 连接
2. **监听消息**: 监听 `message` 事件
3. **识别消息类型**: 根据 `action` 和 `sport_type` 字段识别消息类型
4. **处理特定消息**: 根据不同模块的文档说明处理特定消息
5. **更新界面**: 根据接收到的数据更新大屏显示内容

## 消息类型总览

各测试模块主要使用以下几类消息:

- **状态消息**: 通知系统状态变化，如 `broadcast_analyze_state`、`broadcast_limb_remote_control_status` 等
- **身份消息**: 传递测试者身份信息，如 `broadcast_area_identity`、`broadcast_identity` 等
- **结果消息**: 发送测试结果，如 `broadcast_area_result`、`broadcast_result` 等
- **控制消息**: 接收远程控制命令，如 `screen_navigate_url`、`screen_play_video` 等

## 系统架构

```
┌───────────────┐      ┌───────────────┐
│               │      │               │
│ 测试设备终端   │◀────▶│    服务器     │
│ (相机/传感器)  │      │ (Socket.io)   │
│               │      │               │
└───────────────┘      └───────┬───────┘
                              ▲
                              │
                              ▼
┌───────────────┐      ┌──────────────┐
│               │      │              │
│   遥控设备     │─────▶│   大屏终端    │
│               │      │              │
└───────────────┘      └──────────────┘
```

## 如何使用这些文档

1. 首先阅读 [消息格式规范.md](消息格式规范.md) 了解通用的消息结构和规则
2. 根据项目需求，查阅相应的测试模块文档
3. 参考文档中的接口定义和示例代码，实现大屏端的消息处理逻辑
4. 按照文档中的测试流程和状态转换图，设计大屏端的界面逻辑和状态管理
5. 使用文档中提供的数据结构，正确解析和处理接收到的消息

## 注意事项

1. 所有模块共用同一个 Socket.io 连接，通过 `sport_type` 字段区分不同测试项目
2. 大屏端需要同时处理多种测试项目的消息，建议使用模块化设计
3. 对于实时数据显示，应考虑性能优化，避免界面卡顿
4. 测试结果数据应妥善保存，以便于后续查询和统计
5. 建议实现错误处理和重连机制，提高系统稳定性
6. 大屏端需要验证接收的控制命令是否针对本机，通过检查 `to` 字段

## 更新日志

### v1.0 (2025年4月16日)
- 初始版本发布
- 包含所有测试模块的接口文档
- 提供统一的消息格式规范 