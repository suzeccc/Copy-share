# LAN Clipboard Sync MVP 设计

## 来源
本设计派生自 `clipboard-sync-design.md`。用户要求基于该文档完成目标，因此该文档被视为本次实现的已批准设计来源。

## 目标
创建一个局域网文本剪贴板同步工具。它可以在两台或多台机器上运行，通过 WebSocket 发送剪贴板更新，防止同步循环，支持手动连接 peer，并在连接断开后自动重连。

## 范围
MVP 仅支持文本剪贴板。不支持图片、文件、云端中继、自动设备发现、剪贴板历史、移动端客户端、加密或 PIN 配对。

## 架构
每个进程都会启动一个 WebSocket server，默认端口为 `8765`，也可以启动一个或多个向外连接的 peer client。剪贴板 watcher 会轮询本机文本剪贴板。当本机文本发生变化时，同步引擎会创建 JSON 剪贴板消息并广播给已连接的 peer。当收到远端剪贴板消息时，同步引擎会进行去重，将文本写入本机剪贴板，并抑制下一次 watcher 产生的回声。

## 协议
剪贴板消息与源设计保持一致：

```json
{
  "type": "clipboard",
  "id": "uuid",
  "deviceId": "device-A",
  "timestamp": 1710000000,
  "format": "text",
  "content": "hello world"
}
```

也接受应用层 ping/pong 消息：

```json
{ "type": "ping" }
{ "type": "pong" }
```

## 组件
- `messages`：创建、校验、编码、解码协议消息，并计算内容 hash。
- `sync_engine`：跟踪已见消息 ID、本地/远端最近 hash，以及远端写入后的回声抑制。
- `clipboard`：抽象剪贴板访问；提供 Windows PowerShell 实现和内存实现。
- `websocket`：基于标准库的最小 WebSocket server/client，用于传输文本 JSON 消息。
- `app`：编排 watcher、同步引擎、剪贴板适配器和 peer transport。
- `__main__`：暴露 CLI 参数，包括端口、peer URL/IP、轮询间隔和设备 ID。

## 错误处理
无效 JSON 和不支持的消息类型会被忽略，不会导致同步循环崩溃。peer 连接失败后会按可配置的重连间隔重试。剪贴板读写失败会记录日志，并在后续轮询 tick 中继续重试。

## 测试
单元测试覆盖消息校验、hash 行为、重复消息抑制、远端写入防循环、WebSocket URL 规范化/帧处理，以及 CLI 参数解析。验证还包括 CLI help/import 检查。

## 验收标准
- 两个进程可以通过 `--peer <ip-or-url>` 建立连接。
- 本地文本剪贴板变化会产生 WebSocket 剪贴板消息。
- 远端剪贴板消息只会写入本机文本一次。
- 重复消息 ID 和重复内容不会造成无限循环。
- 向外连接的 peer 断开后可以自动重连。
