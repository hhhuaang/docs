# 人脸 Socket.IO 接口文档 - 控制端

## 目录

- [1. 安装说明](#1-安装说明)
  - [1.1 安装依赖](#11-安装依赖)
  - [1.2 基本连接](#12-基本连接)
- [2. 📤 发送的信令（控制端 → 服务器）](#2--发送的信令控制端--服务器)
  - [2.1 📤 `join` - 加入房间（必需）](#21--join---加入房间必需)
  - [2.2 📤 `control` - 控制命令](#22--control---控制命令)
    - [2.2.1 📤 创建学生 `students_create`](#221--创建学生-students_create)
    - [2.2.2 📤 更新学生 `students_update`](#222--更新学生-students_update)
    - [2.2.3 📤 删除学生 `students_delete`](#223--删除学生-students_delete)
    - [2.2.4 📤 分页查询学生列表 `students_list`](#224--分页查询学生列表-students_list)
    - [2.2.5 📤 按ID获取学生 `student_get_by_id`](#225--按id获取学生-student_get_by_id)
    - [2.2.6 📤 按extra_id获取学生 `student_get_by_extra_id`](#226--按extra_id获取学生-student_get_by_extra_id)
    - [2.2.7 📤 按user_id添加人脸 `faces_create`](#227--按user_id添加人脸-faces_create)
    - [2.2.8 📤 按extra_id添加人脸 `faces_create_by_extra_id`](#228--按extra_id添加人脸-faces_create_by_extra_id)
    - [2.2.9 📤 删除人脸 `faces_delete`](#229--删除人脸-faces_delete)
    - [2.2.10 📤 获取人脸列表 `faces_list`](#2210--获取人脸列表-faces_list)
  - [2.3 📤 `gpu` - 查询GPU列表或发送GPU命令](#23--gpu---查询gpu列表或发送gpu命令)
    - [2.3.1 📤 查询GPU列表](#231--查询gpu列表)
    - [2.3.2 📤 发送GPU命令](#232--发送gpu命令)
- [3. 📥 接收的信令（服务器 → 控制端）](#3--接收的信令服务器--控制端)
  - [3.1 📥 `state_join_gpu` - 被控制端上线通知](#31--state_join_gpu---被控制端上线通知)
  - [3.2 📥 `state_disconnect_gpu` - 被控制端掉线通知](#32--state_disconnect_gpu---被控制端掉线通知)
  - [3.3 📥 `state_list_gpu` - 被控制端列表查询结果](#33--state_list_gpu---被控制端列表查询结果)
  - [3.4 📥 `{action}_report` - 接收设备上报数据](#34--action_report---接收设备上报数据)
    - [3.4.1 📥 事件命名规则](#341--事件命名规则)
    - [3.4.2 📥 接收状态上报](#342--接收状态上报)
    - [3.4.3 📥 接收错误上报](#343--接收错误上报)
    - [3.4.4 📥 接收学生管理结果上报](#344--接收学生管理结果上报)
    - [3.4.5 📥 接收人脸管理结果上报](#345--接收人脸管理结果上报)
    - [3.4.6 📥 接收错误上报](#346--接收错误上报)
- [4. 完整示例](#4-完整示例)
- [5. API 快速参考](#5-api-快速参考)
  - [📤 发送的信令](#-发送的信令)
  - [📥 监听的信令](#-监听的信令)
- [6. 常见问题](#6-常见问题)
- [7. 注意事项](#7-注意事项)
- [8. 技术支持](#8-技术支持)

本文档帮助您快速集成人脸识别/检测的Socket.IO控制功能。控制端可以发送命令控制设备执行，并实时接收设备状态和数据上报。

## 1. 安装说明

### 1.1 安装依赖

```bash
npm install socket.io-client
# 或
yarn add socket.io-client
```

### 1.2 基本连接

```javascript
import io from 'socket.io-client';

// 连接到服务器
const socket = io('ws://127.0.0.1:8080/face', {
  cors: true
});

// 监听连接状态
socket.on('connect', () => {
  console.log('已连接到人脸Socket服务');
  // 连接成功后，立即加入房间
  socket.emit('join', {
    roomId: 'room_001'  // 替换为实际的房间ID
  });
});

socket.on('disconnect', () => {
  console.log('已断开连接');
});
```

---

## 2. 📤 发送的信令（控制端 → 服务器）

**本节说明：控制端主动发送给服务器的信令**

以下信令由控制端主动发送，使用 `socket.emit()` 方法。

### 2.1 📤 `join` - 加入房间（必需）

在开始使用前，必须先加入指定的房间。房间ID必须与被控制端使用的房间ID一致。

```javascript
socket.emit('join', {
  roomId: 'room_001'  // 房间ID，必须与被控制端一致
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `roomId` | string | 是 | 房间ID，必须与被控制端的roomId一致 |

**说明：**
- 加入房间后，如果被控制端已在线，会收到 `state_join_gpu` 事件
- 如果被控制端未在线，会立即收到 `state_disconnect_gpu` 事件
- 发送新的 `join` 事件会自动切换房间

---

### 2.2 📤 `control` - 控制命令

通过 `control` 指令可以发送各种控制命令到设备端。设备端会收到 `{action}_control` 事件。

所有控制事件都遵循统一格式：

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'action_name',  // 动作名称，设备端会收到 'action_name_control' 事件
  data: {                 // 数据对象
    // 具体参数
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `roomId` | string | 是 | 房间ID |
| `action` | string | 是 | 动作名称，设备端会收到 `{action}_control` 事件 |
| `data` | object | 是 | 数据对象，根据不同的 action 包含不同的字段 |

##### 2.2.1 📤 创建学生 `students_create`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'students_create',
  data: {
    extra_id: '20240001',      // 学号/工号（必需）
    name: '张三',               // 姓名（必需）
    person_class: '三年二班',   // 班级（可选）
    grade: '三年级'             // 年级（可选）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `extra_id` | string | 是 | 学号/工号 |
| `name` | string | 是 | 姓名 |
| `person_class` | string | 否 | 班级 |
| `grade` | string | 否 | 年级 |

**上报事件：** `students_create_report`

##### 2.2.2 📤 更新学生 `students_update`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'students_update',
  data: {
    id: 1,                    // 学生ID（必需）
    name: '张三丰',           // 姓名（可选）
    person_class: '武当班',   // 班级（可选）
    grade: '高年级'           // 年级（可选）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `id` | number | 是 | 学生ID |
| `name` | string | 否 | 姓名 |
| `person_class` | string | 否 | 班级 |
| `grade` | string | 否 | 年级 |

**上报事件：** `students_update_report`

##### 2.2.3 📤 删除学生 `students_delete`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'students_delete',
  data: {
    id: 1  // 学生ID（必需）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `id` | number | 是 | 学生ID |

**上报事件：** `students_delete_report`

##### 2.2.4 📤 分页查询学生列表 `students_list`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'students_list',
  data: {
    page: 1,       // 页码（可选，默认1）
    page_size: 10  // 每页数量（可选，默认10）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `page` | number | 否 | 页码，默认 1 |
| `page_size` | number | 否 | 每页数量，默认 10 |

**上报事件：** `students_list_report`

##### 2.2.5 📤 按ID获取学生 `student_get_by_id`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'student_get_by_id',
  data: {
    id: 1  // 学生ID（必需，也可用 student_id）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `id` 或 `student_id` | number | 是 | 学生ID |

**上报事件：** `student_get_by_id_report`

##### 2.2.6 📤 按extra_id获取学生 `student_get_by_extra_id`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'student_get_by_extra_id',
  data: {
    extra_id: 'S12345'  // 学号/工号（必需）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `extra_id` | string | 是 | 学号/工号 |

**上报事件：** `student_get_by_extra_id_report`

##### 2.2.7 📤 按user_id添加人脸 `faces_create`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'faces_create',
  data: {
    user_id: 1,                                    // 用户ID（必需）
    image_base64: '/9j/4AAQSkZJRgABAQEBLAEs...'   // Base64编码的图片（必需）（上传图片分辨率需要必须控制在640*640内，过大的图片会导致socket.io离线）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `user_id` | number | 是 | 用户ID |
| `image_base64` | string | 是 | Base64编码的图片，可包含 `data:image/jpeg;base64,` 前缀 |

**注意事项：**
- 图片中必须只有一张人脸
- 分辨率不低于 112x112 像素
- Base64 格式需正确

**上报事件：** `faces_create_report`

##### 2.2.8 📤 按extra_id添加人脸 `faces_create_by_extra_id`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'faces_create_by_extra_id',
  data: {
    extra_id: 'S12345',                            // 学号/工号（必需）
    image_base64: '/9j/4AAQSkZJRgABAQEBLAEs...'   // Base64编码的图片（必需）（上传图片分辨率必须控制在640*640内，过大的图片会导致socket.io离线）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `extra_id` | string | 是 | 学号/工号 |
| `image_base64` | string | 是 | Base64编码的图片 |

**上报事件：** `faces_create_by_extra_id_report`

##### 2.2.9 📤 删除人脸 `faces_delete`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'faces_delete',
  data: {
    face_id: 101  // 人脸ID（必需）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `face_id` | number | 是 | 人脸ID |

**上报事件：** `faces_delete_report`

##### 2.2.10 📤 获取人脸列表 `faces_list`

```javascript
socket.emit('control', {
  roomId: 'room_001',
  action: 'faces_list',
  data: {
    user_id: 1,      // 用户ID（必需）
    has_feat: false  // 是否返回特征向量（可选，默认false）
  }
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `user_id` | number | 是 | 用户ID |
| `has_feat` | boolean | 否 | 是否返回512维特征向量，默认 false |

**上报事件：** `faces_list_report`

**通用注意事项：**
- 如果被控制端不在线，会收到 `state_disconnect_gpu` 事件
- 处理完成后，设备端会通过 `report` 接口上报结果，控制端会收到 `{action}_report` 事件
- 所有操作都应在设备端检查 `roomId` 是否匹配

---

### 2.3 📤 `gpu` - 查询GPU列表或发送GPU命令

#### 2.3.1 📤 查询GPU列表

```javascript
socket.emit('gpu', {
  action: 'list_gpu'
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `action` | string | 是 | 固定值 `'list_gpu'` |

**说明：**
- 发送后会收到 `state_list_gpu` 事件，包含所有在线的GPU房间ID列表

#### 2.3.2 📤 发送GPU命令

```javascript
socket.emit('gpu', {
  roomId: 'room_001',
  action: 'custom_action',  // 自定义动作
  // ... 其他参数
});
```

**参数说明：**

| 参数名 | 类型 | 必需 | 说明 |
|--------|------|------|------|
| `roomId` | string | 是 | 房间ID |
| `action` | string | 是 | 自定义动作名称（不能是 `'list_gpu'`） |

**说明：**
- 如果被控制端不在线，会收到 `state_disconnect_gpu` 事件
- 设备端会收到 `{action}_gpu` 事件

**完整示例：**

```javascript
// 查询在线设备列表
socket.emit('gpu', {
  action: 'list_gpu'
});

// 发送自定义GPU命令
socket.emit('gpu', {
  roomId: 'room_001',
  action: 'capture',
  savePath: '/path/to/save'
});
```

---

## 3. 📥 接收的信令（服务器 → 控制端）

**本节说明：控制端被动接收来自服务器的信令**

以下信令由控制端被动接收，需要使用 `socket.on()` 方法设置监听器。

### 3.1 📥 `state_join_gpu` - 被控制端上线通知

当被控制端上线时，控制端会收到此事件。

```javascript
socket.on('state_join_gpu', (data) => {
  console.log('被控制端已上线，可以开始控制了');
  // data: { roomId: 'room_001', message: '上线了' }
  enableControlButtons(); // 启用控制按钮
});
```

**数据格式：**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `roomId` | string | 房间ID |
| `message` | string | 固定值 `'上线了'` |

**数据示例：**
```json
{
  "roomId": "room_001",
  "message": "上线了"
}
```

**触发时机：**
- 被控制端上线时
- 控制端发送 `join` 事件时，如果被控制端已在线

---

### 3.2 📥 `state_disconnect_gpu` - 被控制端掉线通知

当被控制端掉线时，控制端会收到此事件。

```javascript
socket.on('state_disconnect_gpu', (data) => {
  console.log('被控制端已掉线');
  // data: { roomId: 'room_001', message: '掉线了' }
  disableControlButtons(); // 禁用控制按钮
  showErrorMessage('被控制端已断开连接，请检查连接');
});
```

**数据格式：**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `roomId` | string | 房间ID |
| `message` | string | 固定值 `'掉线了'` |

**数据示例：**
```json
{
  "roomId": "room_001",
  "message": "掉线了"
}
```

**触发时机：**
- 被控制端断开连接时
- 控制端发送 `control`、`gpu` 事件时，如果被控制端不在线
- 控制端发送 `join` 事件时，如果被控制端不在线

---

### 3.3 📥 `state_list_gpu` - 被控制端列表查询结果

当发送 `gpu` 事件且 `action === 'list_gpu'` 时，会收到此事件。

```javascript
socket.on('state_list_gpu', (data) => {
  console.log('在线GPU列表:', data);
  // data: { roomIds: Set迭代器 }
  // 转换为数组
  const roomIds = Array.from(data.roomIds);
  console.log('在线房间:', roomIds);
  // 更新UI显示在线设备列表
  updateDeviceList(roomIds);
});
```

**数据格式：**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `roomIds` | Set迭代器 | 在线被控制端的房间ID集合的迭代器，需要转换为数组使用 |

**触发时机：**
- 控制端发送 `gpu` 事件且 `action === 'list_gpu'` 时

---

### 3.4 📥 `{action}_report` - 接收设备上报数据

被控制端会在执行操作后通过 `report` 接口上报结果，控制端需要监听对应的事件。

#### 3.4.1 📥 事件命名规则

- 如果上报数据包含 `action` 字段，控制端收到的事件名为 `{action}_report`
- 例如：`action: 'status'` → 控制端收到 `status_report` 事件
- 例如：`action: 'setMode'` → 控制端收到 `setMode_report` 事件
- 例如：`action: 'error'` → 控制端收到 `error_report` 事件
- 如果上报数据不包含 `action` 字段，控制端收到的事件名为 `report`

#### 3.4.2 📥 接收状态上报

```javascript
socket.on('status_report', (data) => {
  console.log('状态更新:', data);
  // data 可能包含：
  // {
  //   roomId: 'room_001',
  //   action: 'status',
  //   data: {
  //     state: 'running',  // idle(空闲), running(运行中), paused(已暂停), error(错误)
  //     camera: 'ok',
  //     ... 其他状态数据
  //   }
  // }
  
  updateStatusDisplay(data.data.state);
});
```

#### 3.4.3 📥 接收错误上报

```javascript
socket.on('error_report', (data) => {
  console.error('错误上报:', data);
  // data 可能包含：
  // {
  //   roomId: 'room_001',
  //   action: 'error',
  //   message: 'camera offline',
  //   errorCode: 'DEVICE_ERROR',
  //   ... 其他错误信息
  // }
  
  showErrorMessage(data.message);
});
```

#### 3.4.4 📥 接收学生管理结果上报

##### 创建学生结果 `students_create_report`

```javascript
socket.on('students_create_report', (data) => {
  console.log('创建学生结果:', data);
  if (data.data.success) {
    console.log('创建成功:', data.data);
    // data.data 包含: { success: true, id, extra_id, name, person_class, grade }
  } else {
    console.error('创建失败:', data.message);
  }
});
```

**成功数据示例：**
```json
{
  "roomId": "room_001",
  "action": "students_create",
  "data": {
    "success": true,
    "id": 1,
    "extra_id": "20240001",
    "name": "张三",
    "person_class": "三年二班",
    "grade": "三年级"
  }
}
```

**失败数据示例：**
```json
{
  "roomId": "room_001",
  "action": "students_create",
  "data": {
    "success": false
  },
  "message": "添加失败"
}
```

##### 更新学生结果 `students_update_report`

```javascript
socket.on('students_update_report', (data) => {
  if (data.data.success) {
    console.log('更新成功');
  } else {
    console.error('更新失败:', data.message);
  }
});
```

##### 删除学生结果 `students_delete_report`

```javascript
socket.on('students_delete_report', (data) => {
  if (data.data.success) {
    console.log('删除成功');
  } else {
    console.error('删除失败:', data.message);
  }
});
```

##### 学生列表结果 `students_list_report`

```javascript
socket.on('students_list_report', (data) => {
  if (data.data.success) {
    console.log('学生列表:', data.data.items);
    console.log('总数:', data.data.total);
    console.log('当前页:', data.data.page);
    // data.data 包含: { success: true, total, page, page_size, items: [...] }
  }
});
```

**数据示例：**
```json
{
  "roomId": "room_001",
  "action": "students_list",
  "data": {
    "success": true,
    "total": 23,
    "page": 1,
    "page_size": 10,
    "items": [
      {
        "id": 1,
        "extra_id": "S12345",
        "name": "张三",
        "person_class": "Class A",
        "grade": "Grade 10"
      }
    ]
  }
}
```

##### 按ID获取学生结果 `student_get_by_id_report`

```javascript
socket.on('student_get_by_id_report', (data) => {
  if (data.data.success) {
    console.log('学生信息:', data.data);
  } else {
    console.error('获取失败:', data.message);
  }
});
```

##### 按extra_id获取学生结果 `student_get_by_extra_id_report`

```javascript
socket.on('student_get_by_extra_id_report', (data) => {
  if (data.data.success) {
    console.log('学生信息:', data.data);
  } else {
    console.error('获取失败:', data.message);
  }
});
```

#### 3.4.5 📥 接收人脸管理结果上报

##### 添加人脸结果 `faces_create_report`

```javascript
socket.on('faces_create_report', (data) => {
  if (data.data.success) {
    console.log('添加人脸成功:', data.data);
    // data.data 包含: { success: true, face_id, user_id, image_path }
  } else {
    console.error('添加人脸失败:', data.message);
    // 可能原因：无人脸、多张人脸、分辨率不够、Base64格式错误等
  }
});
```

**成功数据示例：**
```json
{
  "roomId": "room_001",
  "action": "faces_create",
  "data": {
    "success": true,
    "face_id": 101,
    "user_id": 1,
    "image_path": "/face_images/1_1678886400000.jpg"
  }
}
```

##### 按extra_id添加人脸结果 `faces_create_by_extra_id_report`

```javascript
socket.on('faces_create_by_extra_id_report', (data) => {
  if (data.data.success) {
    console.log('添加人脸成功:', data.data);
  } else {
    console.error('添加人脸失败:', data.message);
  }
});
```

##### 删除人脸结果 `faces_delete_report`

```javascript
socket.on('faces_delete_report', (data) => {
  if (data.data.success) {
    console.log('删除人脸成功');
  } else {
    console.error('删除人脸失败:', data.message);
  }
});
```

##### 人脸列表结果 `faces_list_report`

```javascript
socket.on('faces_list_report', (data) => {
  // 注意：data.data 直接是数组，不包含 success 字段
  const faces = data.data;  // 数组
  console.log('人脸列表:', faces);
  faces.forEach(face => {
    console.log('Face ID:', face.face_id);
    console.log('User ID:', face.user_id);
    console.log('Image Base64:', face.image_base64 ? '已加载' : '未加载');
    // 如果 has_feat=true，face.feat 为512维数组；否则为 null
  });
});
```

**数据示例（数组格式）：**
```json
{
  "roomId": "30fb9b4733d0dcf3e6657c74a1ebd032",
  "action": "faces_list",
  "data": [
    {
      "face_id": 20583,
      "user_id": 1,
      "image_base64": "/9j/4AAQSkZJRgAB6d6/TYNNHxzWh//2Q==",
      "feat": null
    }
  ]
}
```

**字段说明：**
- `face_id` (int): 人脸 ID
- `user_id` (int): 用户 ID
- `image_base64` (string|null): Base64 编码的图片数据，可直接用于前端显示（无需 HTTP 请求），如果文件不存在或读取失败则为 `null`
- `feat` (array|null): 如果请求时 `has_feat=true`，返回 512 维特征向量数组；否则为 `null`

**注意：**
- `data` 字段直接是数组，不包含 `success` 字段
- Socket.IO 返回 `image_base64`（Base64 编码的图片数据），而 REST API 返回 `image_path`（URL）
- 图片读取失败时，`image_base64` 可能为 `null`（不会导致整体失败）

#### 3.4.6 📥 接收错误上报

```javascript
socket.on('error_report', (data) => {
  console.error('错误上报:', data);
  // data 可能包含：
  // {
  //   roomId: 'room_001',
  //   action: 'error',
  //   message: '错误信息',
  //   errorCode: 'ERROR_CODE',
  //   ... 其他错误信息
  // }
  
  showErrorMessage(data.message);
});
```

**通用上报数据格式：**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `roomId` | string | 房间ID |
| `action` | string | 动作名称，对应原控制事件的 action |
| `data` | object 或 array | 结果数据（某些接口返回数组） |
| `data.success` | boolean | 是否成功（**仅对象格式时存在**，数组格式时不存在） |
| `message` | string | 错误信息（可选，通常在失败时提供） |

**触发时机：**
- 设备端发送 `report` 事件时，根据 `action` 字段动态触发对应的事件

**注意事项：**
- **数组格式**：`faces_list_report` 的 `data` 字段直接是数组，不包含 `success` 字段
  - 成功时：`data` 是数组（可能为空数组 `[]`）
  - 失败时：`data` 是包含 `success: false` 的对象，并包含 `message` 字段
- **对象格式**：其他上报的 `data` 字段是对象，且必须包含 `success` 字段
- **`faces_list_report` 特殊说明**：返回的数组中每个元素包含 `image_base64` 字段（Base64 编码的图片数据），可直接用于前端显示，无需 HTTP 请求

---

## 4. 完整示例

以下是一个完整的控制端实现示例：

```javascript
import io from 'socket.io-client';

class FaceController {
  constructor(serverUrl, roomId) {
    this.roomId = roomId;
    this.socket = io(`${serverUrl}/face`, { cors: true });
    this.isGpuOnline = false;
    this.setupListeners();
  }

  setupListeners() {
    // 连接状态
    this.socket.on('connect', () => {
      console.log('已连接到服务器');
      this.joinRoom();
    });

    this.socket.on('disconnect', () => {
      console.log('已断开连接');
      this.isGpuOnline = false;
      this.onDisconnect();
    });

    // ========== 监听的信令 ==========
    
    // 被控制端状态监听
    this.socket.on('state_join_gpu', (data) => {
      console.log('被控制端上线:', data);
      this.isGpuOnline = true;
      this.onGpuOnline();
    });

    this.socket.on('state_disconnect_gpu', (data) => {
      console.log('被控制端掉线:', data);
      this.isGpuOnline = false;
      this.onGpuOffline();
    });

    // 设备列表监听
    this.socket.on('state_list_gpu', (data) => {
      const roomIds = Array.from(data.roomIds);
      this.onDeviceList(roomIds);
    });

    // ========== 学生管理上报监听 ==========
    this.socket.on('students_create_report', (data) => {
      this.onStudentCreateResult(data);
    });

    this.socket.on('students_update_report', (data) => {
      this.onStudentUpdateResult(data);
    });

    this.socket.on('students_delete_report', (data) => {
      this.onStudentDeleteResult(data);
    });

    this.socket.on('students_list_report', (data) => {
      this.onStudentListResult(data);
    });

    this.socket.on('student_get_by_id_report', (data) => {
      this.onStudentGetByIdResult(data);
    });

    this.socket.on('student_get_by_extra_id_report', (data) => {
      this.onStudentGetByExtraIdResult(data);
    });

    // ========== 人脸管理上报监听 ==========
    this.socket.on('faces_create_report', (data) => {
      this.onFaceCreateResult(data);
    });

    this.socket.on('faces_create_by_extra_id_report', (data) => {
      this.onFaceCreateByExtraIdResult(data);
    });

    this.socket.on('faces_delete_report', (data) => {
      this.onFaceDeleteResult(data);
    });

    this.socket.on('faces_list_report', (data) => {
      this.onFaceListResult(data);
    });

    // ========== 其他上报监听 ==========
    this.socket.on('error_report', (data) => {
      this.onError(data);
    });
  }

  // ========== 发送的信令 ==========
  
  // 加入房间
  joinRoom() {
    this.socket.emit('join', {
      roomId: this.roomId
    });
  }

  // 查询设备列表
  queryDeviceList() {
    this.socket.emit('gpu', {
      action: 'list_gpu'
    });
  }

  // ========== 学生管理方法 ==========

  // 创建学生
  createStudent(extraId, name, personClass, grade) {
    if (!this.isGpuOnline) {
      console.warn('被控制端未在线');
      return;
    }
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'students_create',
      data: {
        extra_id: extraId,
        name: name,
        person_class: personClass,
        grade: grade
      }
    });
  }

  // 更新学生
  updateStudent(id, updates) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'students_update',
      data: { id, ...updates }
    });
  }

  // 删除学生
  deleteStudent(id) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'students_delete',
      data: { id }
    });
  }

  // 查询学生列表
  listStudents(page = 1, pageSize = 10) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'students_list',
      data: { page, page_size: pageSize }
    });
  }

  // 按ID获取学生
  getStudentById(id) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'student_get_by_id',
      data: { id }
    });
  }

  // 按extra_id获取学生
  getStudentByExtraId(extraId) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'student_get_by_extra_id',
      data: { extra_id: extraId }
    });
  }

  // ========== 人脸管理方法 ==========

  // 按user_id添加人脸
  createFace(userId, imageBase64) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'faces_create',
      data: {
        user_id: userId,
        image_base64: imageBase64
      }
    });
  }

  // 按extra_id添加人脸
  createFaceByExtraId(extraId, imageBase64) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'faces_create_by_extra_id',
      data: {
        extra_id: extraId,
        image_base64: imageBase64
      }
    });
  }

  // 删除人脸
  deleteFace(faceId) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'faces_delete',
      data: { face_id: faceId }
    });
  }

  // 获取人脸列表
  listFaces(userId, hasFeat = false) {
    if (!this.isGpuOnline) return;
    this.socket.emit('control', {
      roomId: this.roomId,
      action: 'faces_list',
      data: {
        user_id: userId,
        has_feat: hasFeat
      }
    });
  }

  // ========== 其他方法 ==========

  // ========== 回调方法（需要根据业务实现） ==========

  onGpuOnline() {
    console.log('被控制端已上线，可以开始控制了');
  }

  onGpuOffline() {
    console.log('被控制端已掉线');
  }

  onDeviceList(roomIds) {
    console.log('在线设备列表:', roomIds);
  }

  // 学生管理回调
  onStudentCreateResult(data) {
    if (data.data.success) {
      console.log('创建学生成功:', data.data);
    } else {
      console.error('创建学生失败:', data.message);
    }
  }

  onStudentUpdateResult(data) {
    if (data.data.success) {
      console.log('更新学生成功');
    } else {
      console.error('更新学生失败:', data.message);
    }
  }

  onStudentDeleteResult(data) {
    if (data.data.success) {
      console.log('删除学生成功');
    } else {
      console.error('删除学生失败:', data.message);
    }
  }

  onStudentListResult(data) {
    if (data.data.success) {
      console.log('学生列表:', data.data.items);
      console.log('总数:', data.data.total);
    }
  }

  onStudentGetByIdResult(data) {
    if (data.data.success) {
      console.log('学生信息:', data.data);
    } else {
      console.error('获取学生失败:', data.message);
    }
  }

  onStudentGetByExtraIdResult(data) {
    if (data.data.success) {
      console.log('学生信息:', data.data);
    } else {
      console.error('获取学生失败:', data.message);
    }
  }

  // 人脸管理回调
  onFaceCreateResult(data) {
    if (data.data.success) {
      console.log('添加人脸成功:', data.data);
    } else {
      console.error('添加人脸失败:', data.message);
    }
  }

  onFaceCreateByExtraIdResult(data) {
    if (data.data.success) {
      console.log('添加人脸成功:', data.data);
    } else {
      console.error('添加人脸失败:', data.message);
    }
  }

  onFaceDeleteResult(data) {
    if (data.data.success) {
      console.log('删除人脸成功');
    } else {
      console.error('删除人脸失败:', data.message);
    }
  }

  onFaceListResult(data) {
    // 注意：data.data 是数组，不包含 success 字段
    const faces = data.data;
    console.log('人脸列表:', faces);
    faces.forEach(face => {
      console.log('Face ID:', face.face_id);
      console.log('User ID:', face.user_id);
      // image_base64 可直接用于前端显示（Base64 编码的图片数据）
      if (face.image_base64) {
        console.log('Image Base64:', face.image_base64.substring(0, 50) + '...');
      } else {
        console.log('Image Base64: null (图片读取失败)');
      }
      // 如果 has_feat=true，face.feat 为512维数组；否则为 null
      if (face.feat) {
        console.log('Feature vector length:', face.feat.length);
      }
    });
  }

  // 其他回调
  onError(data) {
    console.error('错误上报:', data);
  }

  onDisconnect() {
    // 断开连接时的处理
    console.log('已断开连接');
  }

  // 断开连接
  disconnect() {
    this.socket.disconnect();
  }
}

// 使用示例
const controller = new FaceController('ws://127.0.0.1:8080', 'room_001');

// 实现回调方法
controller.onGpuOnline = () => {
  // 启用控制按钮
  document.getElementById('startBtn').disabled = false;
};

controller.onGpuOffline = () => {
  // 禁用控制按钮
  document.getElementById('startBtn').disabled = true;
  alert('被控制端已断开连接');
};

controller.onDetectResult = (data) => {
  // 更新UI显示检测结果
  updateFaceDetectionUI(data.data.faces);
};

controller.onRecognizeResult = (data) => {
  // 显示识别结果
  displayPersonInfo(data.data);
};

// 使用示例
controller.onStudentCreateResult = (data) => {
  if (data.data.success) {
    alert(`创建学生成功: ${data.data.name}`);
  }
};

controller.onFaceCreateResult = (data) => {
  if (data.data.success) {
    alert(`添加人脸成功, Face ID: ${data.data.face_id}`);
  } else {
    alert(`添加人脸失败: ${data.message}`);
  }
};

// 绑定按钮事件示例
document.getElementById('createStudentBtn').onclick = () => {
  controller.createStudent('S12345', '张三', '三年二班', '三年级');
};

document.getElementById('createFaceBtn').onclick = () => {
  // 获取图片的 base64
  const imageBase64 = getImageBase64();
  controller.createFace(1, imageBase64);
};

document.getElementById('listStudentsBtn').onclick = () => {
  controller.listStudents(1, 10);
};

document.getElementById('queryBtn').onclick = () => {
  controller.queryDeviceList();
};
```

---

## 5. API 快速参考

### 📤 发送的信令

| 事件名称 | 参数 | 说明 |
|---------|------|------|
| `join` | `{ roomId: string }` | 加入房间（必需） |
| `control` | `{ roomId: string, action: string, data: object }` | 控制命令<br/>- `action` - 动作名称（如 `students_create`, `faces_create` 等）<br/>- `data` - 数据对象，根据不同的 action 包含不同的字段 |
| `gpu` | `{ action: 'list_gpu' }` 或 `{ roomId: string, action: string, ...其他参数 }` | 查询GPU列表或发送GPU命令 |

### 📥 监听的信令

| 事件名称 | 数据格式 | 说明 |
|---------|---------|------|
| `state_join_gpu` | `{ roomId: string, message: '上线了' }` | 被控制端上线通知 |
| `state_disconnect_gpu` | `{ roomId: string, message: '掉线了' }` | 被控制端掉线通知 |
| `state_list_gpu` | `{ roomIds: Set迭代器 }` | 被控制端列表查询结果 |
| `{action}_report` | 根据action类型而定 | 接收上报数据（动态事件名）<br/>**学生管理：**<br/>- `students_create_report` - 创建学生结果<br/>- `students_update_report` - 更新学生结果<br/>- `students_delete_report` - 删除学生结果<br/>- `students_list_report` - 学生列表结果<br/>- `student_get_by_id_report` - 按ID获取学生结果<br/>- `student_get_by_extra_id_report` - 按extra_id获取学生结果<br/>**人脸管理：**<br/>- `faces_create_report` - 添加人脸结果<br/>- `faces_create_by_extra_id_report` - 按extra_id添加人脸结果<br/>- `faces_delete_report` - 删除人脸结果<br/>- `faces_list_report` - 人脸列表结果<br/>**其他：**<br/>- `error_report` - 错误上报 |

---

## 6. 常见问题

### 6.1 如何判断被控制端是否在线？

监听 `state_join_gpu` 和 `state_disconnect_gpu` 事件，维护一个在线状态标志。

```javascript
let isGpuOnline = false;

socket.on('state_join_gpu', () => {
  isGpuOnline = true;
});

socket.on('state_disconnect_gpu', () => {
  isGpuOnline = false;
});
```

### 6.2 发送指令时被控制端不在线怎么办？

所有需要与被控制端通信的指令（`control`、`gpu`）在被控制端不在线时，都会收到 `state_disconnect_gpu` 事件。建议：

1. 在发送前检查被控制端是否在线
2. 监听 `state_disconnect_gpu` 事件，提示用户

### 6.3 如何实现自动重连？

```javascript
socket.on('disconnect', () => {
  console.log('连接断开，3秒后重连...');
  setTimeout(() => {
    socket.connect();
  }, 3000);
});

socket.on('connect', () => {
  // 重连后重新加入房间
  socket.emit('join', {
    roomId: 'room_001'
  });
});
```

### 6.4 如何切换房间？

直接发送新的 `join` 事件即可，服务器会自动处理房间切换。

```javascript
socket.emit('join', {
  roomId: 'room_002'  // 新房间ID
});
```

### 6.5 control 指令如何使用？

`control` 指令支持多种预定义的 `action`，主要包括学生管理、人脸管理、人脸匹配等功能。

```javascript
// 创建学生
socket.emit('control', {
  roomId: 'room_001',
  action: 'students_create',
  data: {
    extra_id: 'S12345',
    name: '张三',
    person_class: '三年二班',
    grade: '三年级'
  }
});

// 监听结果上报
socket.on('students_create_report', (data) => {
  if (data.data.success) {
    console.log('创建成功:', data.data);
  } else {
    console.error('创建失败:', data.message);
  }
});
```

详细的功能列表和参数说明请参考第 2.2 节。

### 6.6 如何监听动态的上报事件？

由于上报事件名是动态的（`{action}_report`），可以使用通配符监听或动态注册监听器。

```javascript
// 方法1：动态注册监听器
function handleReport(action, handler) {
  socket.on(`${action}_report`, handler);
}

// 使用
handleReport('setMode', (data) => {
  console.log('设置模式结果:', data);
});

// 方法2：监听所有上报（如果支持）
socket.onAny((eventName, data) => {
  if (eventName.endsWith('_report')) {
    const action = eventName.replace('_report', '');
    console.log(`收到 ${action} 上报:`, data);
  }
});
```

---

## 7. 注意事项

1. **房间ID一致性**：控制端和被控制端必须使用相同的 `roomId` 才能正常通信

2. **连接顺序**：建议先建立连接并加入房间，再执行其他操作

3. **错误处理**：所有操作都应检查被控制端是否在线，收到 `state_disconnect_gpu` 时需要提示用户

4. **事件命名规则**：
   - 控制命令：发送 `control` 事件（`action` 为自定义值）
   - 上报数据：如果上报数据包含 `action` 字段，控制端收到 `{action}_report` 事件

5. **重连机制**：建议实现自动重连机制，断开后重新发送 `join` 事件

6. **房间切换**：发送新的 `join` 事件会自动切换房间

7. **协议说明**：Socket.IO 客户端通常使用 `http://` 或 `https://` 协议，库会自动处理 WebSocket 升级。如果使用 `ws://` 遇到连接问题，可改回 `http://127.0.0.1:8080/face`

8. **数组返回格式**：`faces_list_report` 的 `data` 字段是数组，不是对象。处理时需要注意：
   ```javascript
   // 错误：data.data 是数组，不是对象，没有 success 字段
   if (data.data.success) { ... }
   
   // 正确：直接使用数组
   const faces = data.data;  // 数组
   faces.forEach(face => { ... });
   ```
   
   **注意**：`faces_list_report` 返回的数组中，每个元素包含 `image_base64` 字段（Base64 编码的图片数据），可直接用于前端显示，无需 HTTP 请求。这与 REST API 返回的 `image_path`（URL）不同。

9. **图片格式要求**：添加人脸时，Base64 编码的图片需满足以下要求：
   - 图片中必须只有一张人脸
   - 分辨率不低于 112x112 像素
   - Base64 格式需正确（可包含 `data:image/jpeg;base64,` 前缀）

---

## 8. 技术支持

如有问题，请参考服务器实现：`src/getaway/face.gateway.ts`
