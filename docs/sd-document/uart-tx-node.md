# UART 通信节点二次开发（uart_tx_node）

## 项目概述

- 工作包名称：`uartforstm`
- 节点名称：`uart_tx_node`
- 用途：订阅 MAVROS 状态与 RC 输入，打包后通过 UART 发送给 MCU。

## 串口参数

- 默认端口：`/dev/ttyS3`（可由 `port` 参数覆盖）
- 波特率：`115200`
- 数据位：`8`
- 停止位：`1`
- 校验位：无
- 流控：无

## 节点功能

- 订阅 `/mavros/state`
- 订阅 `/mavros/rc/in`
- 解析连接状态、解锁状态、飞行模式
- 解析 RC 第 6 通道作为急停状态
- 将状态打包为单字节并串口发送

## 数据协议

### 单字节位定义

| 位索引 | 位名称 | 描述 | 取值说明 |
| :-- | :-- | :-- | :-- |
| bit0-2 | 预留位 | 预留 | 固定为 0 |
| bit3 | 连接状态 | 飞控连接 | `0` 未连接，`1` 已连接 |
| bit4 | 解锁状态 | 飞行器解锁 | `0` 未解锁，`1` 已解锁 |
| bit5 | RC 通道 6 状态 | 急停按钮 | `0` 未触发（1050），`1` 触发（1950） |
| bit6 | 模式低位 | 飞行模式编码位 0 | 与 bit7 共同编码 |
| bit7 | 模式高位 | 飞行模式编码位 1 | 与 bit6 共同编码 |

### 飞行模式映射

| 飞行模式 | bit7 | bit6 | 编码 |
| :-- | :-- | :-- | :-- |
| `POSCTL` | 0 | 1 | `01` |
| `ALTCTL` | 1 | 0 | `10` |
| `OFFBOARD` | 1 | 1 | `11` |
| 其他模式 | 0 | 0 | `00` |

说明：仅在 `have_state == true` 时构造并发送数据包；未连接飞控时发送 `0x00`。

## 参数配置

- `port`（string，默认 `"/dev/ttyS3"`）：串口设备路径，如 `/dev/ttyUSB0`、`/dev/ttyS1`。

## 日志与调试

- `INFO`：串口打开成功、数据发送成功。
- `THROTTLED INFO`：周期状态（连接状态、RC 值）。
- `WARN`：写串口失败、RC 数据长度不足。
- `ERROR`：串口未打开时发送数据。
- `FATAL`：启动时串口打开失败。

## API 参考

### 订阅话题

| 主题 | 消息类型 | 说明 |
| :-- | :-- | :-- |
| `/mavros/state` | `mavros_msgs/State` | 飞控连接、解锁、模式状态 |
| `/mavros/rc/in` | `mavros_msgs/RCIn` | 遥控器通道值（主要用第 6 通道） |

### 内部串口发送函数

```cpp
bool sendSerial(const std::string &data);
bool sendSerialByte(uint8_t b);
bool sendSerialInt32(int32_t value, bool little_endian = true);
bool sendSerialIntAscii(int32_t value, bool with_newline = true);
```
