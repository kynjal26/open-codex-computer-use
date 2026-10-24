## [2026-10-24 12:00] | Task: 修复网页标签点击与命中测试坐标

### Execution Context

- Agent ID: Claude Code
- Base Model: claude-opus-5-5
- Runtime: Claude Code CLI

### User Query

> 修复点击 Search Console 报告页中的「网页 / 国家」等标签时，工具返回成功但标签没有切换的问题。

### Changes Overview

- `click` 的 `auto` 路径：点击前记录 Web 内容中标签 / 单选按钮的选中状态；AX 序列或 app_post 兜底执行后，若该元素在 0.4 秒内仍未选中，则改用窗口定向的 `sky_click` 重试一次。
- `hitTestElement`：调用方传入的是窗口局部坐标（与 `ElementRecord.localFrame` 一致），此前却按截图像素换算。截图被缩小（默认长边 1280）时命中位置偏移，`include_screenshot=false` 时直接报错；现改为 `windowPointToGlobalPoint`。

### Design Intent

Search Console 的 Material 标签接受 AXPress 也接受 `postToPid` 点击事件，但都不会切换；`sky_click` 能可靠选中。只有在点击后可以验证「选中状态仍未改变」时才重试，其他元素和路径的行为不变。命中测试坐标统一使用窗口局部点，与两处调用方的语义一致。

### Files Modified

- `packages/OpenComputerUseKit/Sources/OpenComputerUseKit/ComputerUseService.swift`
- `docs/releases/feature-release-notes.md`

### Validation

- Chrome 中 Search Console 性能报告：对「查询 / 网页 / 国家」标签使用默认 `click`（无 `click_method`、`include_screenshot=false` 的快照）均正确切换，表格内容随之更新。
- `swift build` 与 `OpenComputerUseKitTests` 通过。
