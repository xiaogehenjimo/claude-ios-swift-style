# 线程使用规范

> 引用自 `SKILL.md` 第十一章。UI 操作严格主线程，避免 `main.sync`。

## 用法

```swift
// 切换到全局队列（子线程）
JacoThread.runOnGlobalQueue {
    // 后台处理
    JacoThread.runOnMain {
        // 回到主线程更新 UI
    }
}

// 标准 GCD（也可用）
DispatchQueue.global().async {
    DispatchQueue.main.async {
        // UI 更新
    }
}
```

## 规则

- UI 操作必须在主线程
- 网络请求用 `HTTPRequest`（主线程回调）或 `HTTPRequestInChildThread`（子线程回调）
- 避免 `DispatchQueue.main.sync`（死锁风险）
