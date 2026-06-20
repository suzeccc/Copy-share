# LAN Clipboard Sync MVP 实施计划

> **给 agentic worker：** 必须使用子技能：推荐 `superpowers:subagent-driven-development`，也可使用 `superpowers:executing-plans`，逐项执行本计划。步骤使用 checkbox（`- [ ]`）语法跟踪。

**目标：** 构建一个无第三方依赖的 Python MVP，通过局域网 WebSocket 同步文本剪贴板。

**架构：** 应用是一个 Python CLI，由协议消息、同步决策、剪贴板访问、WebSocket transport 和运行时编排几个小模块组成。它会在 `8765` 端口启动 server，可选连接 peer，监听文本剪贴板变化，并通过消息 ID 和内容 hash 抑制远端写入回声。

**技术栈：** Python 3.14 标准库、`asyncio`、`unittest`、JSON 协议、Windows PowerShell 剪贴板命令。

---

## 文件结构
- 创建 `lan_clipboard_sync/__init__.py`：包元数据。
- 创建 `lan_clipboard_sync/messages.py`：协议 dataclass 和辅助函数。
- 创建 `lan_clipboard_sync/sync_engine.py`：去重和防循环逻辑。
- 创建 `lan_clipboard_sync/clipboard.py`：剪贴板抽象和适配器。
- 创建 `lan_clipboard_sync/websocket.py`：最小 WebSocket server/client。
- 创建 `lan_clipboard_sync/app.py`：watcher 和网络编排。
- 创建 `lan_clipboard_sync/__main__.py`：CLI 入口。
- 创建 `tests/test_messages.py`：消息和 hash 测试。
- 创建 `tests/test_sync_engine.py`：同步行为测试。
- 创建 `tests/test_websocket.py`：URL 和帧测试。
- 创建 `tests/test_cli.py`：CLI parser 测试。
- 创建 `README.md`：使用和验证说明。

### 任务 1：协议消息
**文件：**
- 创建：`tests/test_messages.py`
- 创建：`lan_clipboard_sync/messages.py`

- [x] **步骤 1：编写消息创建和 hash 的失败测试**

运行：`python -m unittest tests.test_messages -v`
预期：失败，因为 `lan_clipboard_sync.messages` 不存在。

- [x] **步骤 2：最小实现 `messages.py`**

定义 `ClipboardMessage`、`content_hash`、`make_clipboard_message`、`parse_message`、`encode_message` 和 `decode_message`。

- [x] **步骤 3：运行协议测试**

运行：`python -m unittest tests.test_messages -v`
预期：通过。

### 任务 2：同步引擎
**文件：**
- 创建：`tests/test_sync_engine.py`
- 创建：`lan_clipboard_sync/sync_engine.py`
- 修改：`lan_clipboard_sync/clipboard.py`

- [x] **步骤 1：编写本地变化和远端消息行为的失败测试**

运行：`python -m unittest tests.test_sync_engine -v`
预期：失败，因为 `lan_clipboard_sync.sync_engine` 不存在。

- [x] **步骤 2：最小实现同步引擎**

定义 `SyncEngine.observe_local_text`、`SyncEngine.apply_remote_message` 和 `SyncEngine.should_suppress_watcher_echo`。

- [x] **步骤 3：运行同步引擎测试**

运行：`python -m unittest tests.test_sync_engine -v`
预期：通过。

### 任务 3：WebSocket Transport
**文件：**
- 创建：`tests/test_websocket.py`
- 创建：`lan_clipboard_sync/websocket.py`

- [x] **步骤 1：编写 URL 规范化和 WebSocket 帧 round-trip 的失败测试**

运行：`python -m unittest tests.test_websocket -v`
预期：失败，因为 `lan_clipboard_sync.websocket` 不存在。

- [x] **步骤 2：实现最小 WebSocket 辅助函数**

定义 `normalize_peer_url`、帧编码/解码辅助函数、server 类和自动重连 peer client。

- [x] **步骤 3：运行 WebSocket 测试**

运行：`python -m unittest tests.test_websocket -v`
预期：通过。

### 任务 4：CLI 和应用编排
**文件：**
- 创建：`tests/test_cli.py`
- 创建：`lan_clipboard_sync/app.py`
- 创建：`lan_clipboard_sync/__main__.py`
- 修改：`lan_clipboard_sync/clipboard.py`

- [x] **步骤 1：编写 parser 默认值和 peer 处理的失败测试**

运行：`python -m unittest tests.test_cli -v`
预期：失败，因为 `build_parser` 不存在。

- [x] **步骤 2：实现 app 和 CLI**

创建一个 async app：启动 WebSocket server，启动自动重连 peer，轮询剪贴板，并广播本地更新。

- [x] **步骤 3：运行 CLI 测试**

运行：`python -m unittest tests.test_cli -v`
预期：通过。

### 任务 5：文档和完整验证
**文件：**
- 创建：`README.md`
- 修改：`progress.md`
- 修改：`task_plan.md`

- [x] **步骤 1：编写使用文档**

记录无安装运行方式、双设备运行命令、限制和测试命令。

- [x] **步骤 2：运行完整验证**

运行：`python -m unittest discover -v`
预期：通过。

运行：`python -m lan_clipboard_sync --help`
预期：退出码 0，并显示 CLI help。

- [x] **步骤 3：更新计划文件**

只有在测试和 help 检查通过后，才将所有阶段标记为完成。

### 任务 6：文档中文化
**文件：**
- 修改：`README.md`
- 修改：`task_plan.md`
- 修改：`findings.md`
- 修改：`progress.md`
- 修改：`docs/superpowers/specs/2026-06-20-lan-clipboard-sync-design.md`
- 修改：`docs/superpowers/plans/2026-06-20-lan-clipboard-sync.md`

- [x] **步骤 1：将生成的 Markdown 文档改为中文**

保留命令、路径、协议字段和代码标识符；其余说明文字使用中文。
