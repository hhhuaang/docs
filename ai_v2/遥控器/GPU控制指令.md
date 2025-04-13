# Socket.io 遥控器GPU控制指令文档

本文档描述了基于 Socket.io 的遥控器与GPU主机之间的控制指令和响应，用于启动和停止体育测试项目。服务端地址为 `wss://gs.classtao.cn/all`，采用 WebSocket 协议。

当前日期：2025年4月16日  
文档版本：1.0  

---

## 目录

- [Socket.io 遥控器GPU控制指令文档](#socketio-遥控器gpu控制指令文档)
  - [目录](#目录)
  - [1. 接口概述](#1-接口概述)
  - [2. 控制指令](#2-控制指令)
    - [2.1 启动体育项目检测](#21-启动体育项目检测)
    - [2.2 停止体育项目检测](#22-停止体育项目检测)
  - [3. 响应消息](#3-响应消息)
    - [3.1 体育项目启动响应](#31-体育项目启动响应)
    - [3.2 体育项目停止响应](#32-体育项目停止响应)
  - [4. 使用示例](#4-使用示例)
    - [4.1 发送启动指令](#41-发送启动指令)
    - [4.2 发送停止指令](#42-发送停止指令)
    - [4.3 处理响应消息](#43-处理响应消息)
  - [5. 注意事项](#5-注意事项)
  - [6. 更新日志](#6-更新日志)
    - [v1.0 (2025年4月16日)](#v10-2025年4月16日)

---

## 1. 接口概述

遥控器与GPU主机之间的通信基于 Socket.io 协议，通过向GPU主机发送控制指令来启动和停止各种体育测试项目。

## 2. 控制指令

### 2.1 启动体育项目检测
- **描述**: 在指定摄像头上启动某个体育项目的检测。
- **事件**: `message`
- **请求格式**:
  ```json
  {
    "action": "sport_start",
    "from": "control_web",
    "to": ["5e110d3a9f236c7de393856bb894b6b7"], // gpu主机硬件ID（gpu_hardware_id）
    "time": 1742569707902,
    "data": {
      "schoolId": "677e4cf2d711833b9a143096",
      "cameraId": "67c85653a26360aced80129d",
      "sportId": "standing_long_jump", // 对应支持的体育项目ID
      "sportMode": 1 // 运动模式，1表示体测模式
    }
  }
  ```
- **参数说明**:
  - `schoolId`: 学校ID，标识该请求所属的学校
  - `cameraId`: 摄像头ID，指定使用哪个摄像头进行检测
  - `sportId`: 体育项目ID，对应支持的体育项目列表中的ID
  - `sportMode`: 运动模式，不同的数值代表不同的检测模式

- **运动模式字典**:
  | 模式值 | 模式名称 | 描述 |
  |-------|---------|------|
  | 1 | 体测 | 体育测试模式，用于学生体育素质测评 |
  | 2 | 体锻 | 体育锻炼模式，用于日常锻炼活动 |
  | 3 | 体考 | 体育考试模式，用于正式体育考核 |
  | 4 | 体教 | 体育教学模式，用于体育课程教学 |

- **原始数据示例**:
  ```
  42/all,["message",{"action":"sport_start","from":"control_web","to":["5e110d3a9f236c7de393856bb894b6b7"],"time":1742569707902,"data":{"schoolId":"677e4cf2d711833b9a143096","cameraId":"67c85653a26360aced80129d","sportId":"standing_long_jump","sportMode":1}}]
  ```

### 2.2 停止体育项目检测
- **描述**: 在指定摄像头上停止当前体育项目的检测。
- **事件**: `message`
- **请求格式**:
  ```json
  {
    "action": "sport_stop",
    "from": "control_web",
    "to": ["5e110d3a9f236c7de393856bb894b6b7"], // gpu_hardware_id
    "time": 1742370038670,
    "data": {
      "cameraId": "67c85653a26360aced80129d"
    }
  }
  ```
- **原始数据示例**:
  ```
  42/all,["message",{"action":"sport_stop","from":"control_web","to":["5e110d3a9f236c7de393856bb894b6b7"],"time":1742370038670,"data":{"cameraId":"67c85653a26360aced80129d"}}]
  ```

## 3. 响应消息

GPU主机会通过 `message` 事件响应遥控器的控制指令。

### 3.1 体育项目启动响应
- **Action**: `sport_start_response`
- **响应格式**:
  ```json
  {
    "action": "sport_start_response",
    "from": "QIGqxpw7h_l1_yRDAAEi",
    "to": ["P5ZO0e-DMn8FA8PCAAEk"],
    "time": "2025-03-19T07:40:49.024544+00:00Z",
    "status": "success",
    "message": "开始运动成功",
    "data": {
      "sportId": "standing_long_jump",
      "cameraId": "67c85653a26360aced80129d"
    }
  }
  ```
- **原始数据示例**:
  ```
  42/all,["message",{"action":"sport_start_response","to":["P5ZO0e-DMn8FA8PCAAEk"],"status":"success","message":"开始运动成功","data":{"sportId":"standing_long_jump","cameraId":"67c85653a26360aced80129d"},"from":"QIGqxpw7h_l1_yRDAAEi","time":"2025-03-19T07:40:49.024544+00:00Z"}]
  ```

### 3.2 体育项目停止响应
- **Action**: `sport_stop_response`
- **响应格式**:
  ```json
  {
    "action": "sport_stop_response",
    "from": "QIGqxpw7h_l1_yRDAAEi",
    "to": ["P5ZO0e-DMn8FA8PCAAEk"],
    "time": "2025-03-19T07:40:38.733726+00:00Z",
    "status": "success",
    "message": "停止运动成功",
    "data": {
      "sportId": "standing_long_jump",
      "cameraId": "67c85653a26360aced80129d"
    }
  }
  ```
- **原始数据示例**:
  ```
  42/all,["message",{"action":"sport_stop_response","to":["P5ZO0e-DMn8FA8PCAAEk"],"status":"success","message":"停止运动成功","data":{"sportId":"standing_long_jump","cameraId":"67c85653a26360aced80129d"},"from":"QIGqxpw7h_l1_yRDAAEi","time":"2025-03-19T07:40:38.733726+00:00Z"}]
  ```

## 4. 使用示例

### 4.1 发送启动指令

```javascript
// 启动体育项目
function startSport(gpuHardwareId, cameraId, sportId, sportMode) {
  socket.emit("message", {
    action: "sport_start",
    from: "control_web",
    to: [gpuHardwareId], // GPU主机硬件ID
    time: Date.now(),
    data: {
      schoolId: "677e4cf2d711833b9a143096",
      cameraId: cameraId,
      sportId: sportId,
      sportMode: sportMode
    }
  });
}

// 调用示例
startSport(
  "5e110d3a9f236c7de393856bb894b6b7", // GPU主机硬件ID
  "67c85653a26360aced80129d",         // 摄像头ID
  "standing_long_jump",               // 体育项目ID
  1                                   // 运动模式：体测模式
);
```

### 4.2 发送停止指令

```javascript
// 停止体育项目
function stopSport(gpuHardwareId, cameraId) {
  socket.emit("message", {
    action: "sport_stop",
    from: "control_web",
    to: [gpuHardwareId], // GPU主机硬件ID
    time: Date.now(),
    data: {
      cameraId: cameraId
    }
  });
}

// 调用示例
stopSport(
  "5e110d3a9f236c7de393856bb894b6b7", // GPU主机硬件ID
  "67c85653a26360aced80129d"          // 摄像头ID
);
```

### 4.3 处理响应消息

```javascript
// 监听响应消息
socket.on("message", (message) => {
  // 处理消息格式
  let msgData;
  if (Array.isArray(message) && message.length === 2) {
    const [eventName, eventData] = message;
    if (eventName === "message" && eventData) {
      msgData = eventData;
    }
  } else if (message.action && message.data) {
    msgData = message;
  }
  
  if (!msgData) return;
  
  // 处理不同类型的响应
  switch (msgData.action) {
    case "sport_start_response":
      handleSportStartResponse(msgData);
      break;
      
    case "sport_stop_response":
      handleSportStopResponse(msgData);
      break;
  }
});

// 处理启动响应
function handleSportStartResponse(response) {
  const { status, message, data } = response;
  
  if (status === "success") {
    console.log(`成功启动${data.sportId}项目，摄像头ID: ${data.cameraId}`);
    // 更新UI状态为"运行中"
    updateUIStatus(data.cameraId, "running");
  } else {
    console.error(`启动项目失败: ${message}`);
    // 显示错误信息
    showErrorMessage(message);
  }
}

// 处理停止响应
function handleSportStopResponse(response) {
  const { status, message, data } = response;
  
  if (status === "success") {
    console.log(`成功停止${data.sportId}项目，摄像头ID: ${data.cameraId}`);
    // 更新UI状态为"已停止"
    updateUIStatus(data.cameraId, "stopped");
  } else {
    console.error(`停止项目失败: ${message}`);
    // 显示错误信息
    showErrorMessage(message);
  }
}
```

## 5. 注意事项

1. **GPU硬件ID**: 确保使用正确的GPU主机硬件ID（`gpu_hardware_id`），这是控制指令的目标。
2. **摄像头ID**: 确保摄像头ID是有效的，并且与指定的GPU主机关联。
3. **运动项目ID**: 使用有效的体育项目ID，可通过服务器查询获取支持的项目列表。
4. **错误处理**: 添加适当的错误处理机制，特别是网络错误和响应超时。
5. **连接状态检查**: 在发送控制指令前，确保Socket.io连接处于连接状态。
6. **合理的超时机制**: 为指令执行设置合理的超时时间，在超时后给出用户提示。
7. **并发控制**: 避免在同一摄像头上同时启动多个体育项目，这可能导致意外行为。

## 6. 更新日志

### v1.0 (2025年4月16日)
- 初始版本发布
- 支持启动和停止体育项目的控制指令
- 提供完整的接口使用示例 