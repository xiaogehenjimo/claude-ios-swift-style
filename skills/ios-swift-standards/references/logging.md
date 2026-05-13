# 日志打印规范

> 引用自 `SKILL.md` 第二章。所有项目内日志统一走以下接口，不要裸用 `print()`。

## 业务日志（持久化，生产可查）

```swift
// info 级别 — 常规业务流程
Logger_i(.home, "首页请求发起 isFirstLoad:\(isFirstLoad)")
Logger_i(.network, "[网络] 状态变化 \(status)")
Logger_i(.live, "[直播] 进入房间 lid:\(lid)")
Logger_i(.common, "通用业务事件")
Logger_i(.homeStream, "[流] 开始播放")

// error 级别 — 错误、异常
Logger_e(.network, "[网络] 请求失败 error:\(error)")

// warning 级别 — 警告
Logger_w(.common, "非预期状态")
```

## 调试日志（仅 Debug/Profile 生效，Release 自动关闭）

```swift
// 单行调试日志优先使用，Release 下是 no-op
JacoSafeDebugLog("调试信息")

// 多行代码情况下用 JacoSafeDebug 包裹
JacoSafeDebug {
    debugPrint("调试信息：\(someValue)")
}
```

## 规则

- **不要**在正式代码中裸用 `print()`，统一使用 `Logger_i` / `JacoSafeDebug` / `JacoSafeDebugLog`
- 日志 category 选最匹配的：`.common` / `.home` / `.network` / `.live` / `.homeStream`
- 关键路径必须打 `Logger_i`，调试辅助信息用 `JacoSafeDebug` / `JacoSafeDebugLog`
