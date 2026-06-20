# LAN Clipboard Sync UI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the approved A+C hybrid UI: a Tkinter desktop console plus a compact status window for LAN Clipboard Sync.

**Architecture:** Keep the sync core unchanged. Add a small UI state model, a controller that owns start/pause/stop intent and device-address normalization, and a Tkinter view that renders state and delegates actions. The first version uses only Python standard library APIs so it can run in the current no-dependency workspace.

**Tech Stack:** Python 3.14 standard library, `unittest`, `tkinter`, existing `lan_clipboard_sync` package.

---

## File Structure
- Create `lan_clipboard_sync/ui/__init__.py`: UI package marker and public exports.
- Create `lan_clipboard_sync/ui/state.py`: dataclasses for devices, log events, and app state.
- Create `lan_clipboard_sync/ui/controller.py`: UI controller, device URL normalization, lifecycle events, and log collection.
- Create `lan_clipboard_sync/ui/tk_app.py`: Tkinter main window and compact status window.
- Create `lan_clipboard_sync/ui/__main__.py`: `python -m lan_clipboard_sync.ui` entrypoint.
- Create `tests/test_ui_state.py`: state-model tests.
- Create `tests/test_ui_controller.py`: controller behavior tests.
- Modify `README.md`: add UI run command and first-version limits.
- Modify `progress.md`: record UI implementation progress and verification.

### Task 1: UI State Model
**Files:**
- Create: `tests/test_ui_state.py`
- Create: `lan_clipboard_sync/ui/__init__.py`
- Create: `lan_clipboard_sync/ui/state.py`

- [x] **Step 1: Write failing state tests**

Create tests that assert:

```python
import unittest

from lan_clipboard_sync.ui.state import PeerStatus, SyncUiState


class UiStateTests(unittest.TestCase):
    def test_default_state_matches_stopped_ui(self) -> None:
        state = SyncUiState()
        self.assertFalse(state.running)
        self.assertEqual(state.status_text, "未启动")
        self.assertEqual(state.connected_count, 0)
        self.assertEqual(state.listen_port, 8765)

    def test_peer_status_updates_connected_count(self) -> None:
        state = SyncUiState()
        state.upsert_peer(PeerStatus(name="device-a", address="192.168.1.12", state="online"))
        state.upsert_peer(PeerStatus(name="office-pc", address="192.168.1.31", state="reconnecting"))
        self.assertEqual(state.connected_count, 1)
        self.assertEqual(state.peers[0].label, "device-a  192.168.1.12")

    def test_logs_keep_latest_entries_first(self) -> None:
        state = SyncUiState(max_logs=2)
        state.add_log("server started")
        state.add_log("device connected")
        state.add_log("duplicate suppressed")
        self.assertEqual([entry.message for entry in state.logs], ["duplicate suppressed", "device connected"])
```

Run: `python -m unittest tests.test_ui_state -v`
Expected: FAIL because `lan_clipboard_sync.ui.state` does not exist.

- [x] **Step 2: Implement `state.py` minimally**

Define:
- `PeerStatus(name: str, address: str, state: str)`
- `UiLogEntry(message: str, level: str = "info")`
- `SyncUiState(...)` with fields for running, paused, listen_port, peers, logs, counters, and methods `upsert_peer`, `add_log`, `set_running`, `set_paused`.

- [x] **Step 3: Run state tests**

Run: `python -m unittest tests.test_ui_state -v`
Expected: PASS.

### Task 2: UI Controller
**Files:**
- Create: `tests/test_ui_controller.py`
- Create: `lan_clipboard_sync/ui/controller.py`

- [x] **Step 1: Write failing controller tests**

Create tests that assert:

```python
import unittest

from lan_clipboard_sync.ui.controller import SyncUiController


class UiControllerTests(unittest.TestCase):
    def test_start_updates_state_and_normalizes_peers(self) -> None:
        controller = SyncUiController(device_id="device-a")
        controller.start(port=9000, peers=["192.168.1.20"])
        self.assertTrue(controller.state.running)
        self.assertEqual(controller.state.listen_port, 9000)
        self.assertEqual(controller.configured_peers, ["ws://192.168.1.20:9000/"])
        self.assertIn("同步服务已启动", controller.state.logs[0].message)

    def test_pause_and_resume_update_state(self) -> None:
        controller = SyncUiController(device_id="device-a")
        controller.start(port=8765, peers=[])
        controller.pause()
        self.assertTrue(controller.state.paused)
        self.assertEqual(controller.state.status_text, "已暂停")
        controller.resume()
        self.assertFalse(controller.state.paused)
        self.assertEqual(controller.state.status_text, "未连接设备")

    def test_invalid_peer_is_logged_without_starting(self) -> None:
        controller = SyncUiController(device_id="device-a")
        controller.start(port=8765, peers=["wss://example.test"])
        self.assertFalse(controller.state.running)
        self.assertIn("设备地址无效", controller.state.logs[0].message)
```

Run: `python -m unittest tests.test_ui_controller -v`
Expected: FAIL because `lan_clipboard_sync.ui.controller` does not exist.

- [x] **Step 2: Implement `controller.py` minimally**

Define `SyncUiController` with:
- constructor accepting `device_id`
- `start(port: int, peers: list[str])`
- `pause()`
- `resume()`
- `stop()`
- `add_peer(address: str)`
- `configured_peers` list

The first controller version does not need to launch the real async sync app inside tests. It prepares a stable lifecycle/state boundary for the Tkinter UI and can be wired to runtime later in the same module.

- [x] **Step 3: Run controller tests**

Run: `python -m unittest tests.test_ui_controller -v`
Expected: PASS.

### Task 3: Tkinter Main Window and Compact Window
**Files:**
- Create: `lan_clipboard_sync/ui/tk_app.py`
- Create: `lan_clipboard_sync/ui/__main__.py`

- [x] **Step 1: Implement Tkinter UI using the controller**

Build:
- `SyncDesktopUi`
- left sidebar with app name, port display, and navigation labels
- top action row with “开始同步”, “暂停/恢复”, “断开全部”, and “紧凑状态”
- metrics row for online devices, latest latency, sync count, duplicate blocks
- device list area
- manual device address input
- log list
- compact status window opened from the main window

- [x] **Step 2: Add UI entrypoint**

`lan_clipboard_sync/ui/__main__.py` should call `run_ui()`.

Run: `python -m lan_clipboard_sync.ui --help`
Expected: exit code 0 and options for `--device-id`, `--port`, and `--peer`.

- [x] **Step 3: Manual launch check**

Run: `python -m lan_clipboard_sync.ui --device-id ui-preview`
Expected: a Tkinter window opens. If the environment is headless or GUI launch is blocked, record the limitation and keep automated tests as the verification baseline.

### Task 4: Documentation and Verification
**Files:**
- Modify: `README.md`
- Modify: `progress.md`

- [x] **Step 1: Update README UI section**

Add:

```powershell
python -m lan_clipboard_sync.ui
python -m lan_clipboard_sync.ui --device-id device-a --peer 192.168.1.20
```

Document that the first UI version uses Tkinter and a compact status window rather than a full system tray implementation.

- [x] **Step 2: Run all tests**

Run: `python -m unittest discover -v`
Expected: existing 16 tests plus new UI tests pass.

Run: `python -m lan_clipboard_sync.ui --help`
Expected: exit code 0.

- [x] **Step 3: Update progress**

Record exact verification output in `progress.md`. Commit is not applicable because this workspace is not a git repository.

## Self-Review
- Spec coverage: state model, controller, desktop console, compact status window, run command, README, and tests are covered.
- Placeholder scan: no unresolved placeholder markers or incomplete steps.
- Type consistency: `PeerStatus`, `UiLogEntry`, `SyncUiState`, and `SyncUiController` names are used consistently.
