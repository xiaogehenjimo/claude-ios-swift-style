# 错误处理 / Error 枚举

> 引用自 `SKILL.md`。配合 `best-practices.md` 中"禁止 assert / fatalError"一起看。

## 一、定义 Error：用 enum 实现 Error

```swift
enum JacoRoomError: Error {
    case invalidRoomId
    case notLoggedIn
    case roomFull(currentCount: Int, maxCount: Int)
    case network(underlying: Error)
    case server(code: Int, message: String)
}

// 提供 localizedDescription，方便日志和 UI 展示
extension JacoRoomError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .invalidRoomId:                  return "房间 ID 非法"
        case .notLoggedIn:                    return "请先登录"
        case .roomFull(let cur, let max):     return "房间已满（\(cur)/\(max)）"
        case .network(let e):                 return "网络异常：\(e.localizedDescription)"
        case .server(let code, let msg):      return "服务异常 \(code)：\(msg)"
        }
    }
}
```

规则：
- 每个模块一个 `JacoXxxError`，文件名 `JacoXxxError.swift`
- case 名称用业务语义，关联值带上排查需要的上下文（错误码、count、underlying error）
- 必须实现 `LocalizedError` 或在调用方手动产出文案，禁止 `\(error)` 直接给用户看

## 二、错误传递选型

| 场景 | 用法 |
|---|---|
| 函数无返回值、需告知失败原因 | `throws` |
| 函数有返回值，且失败时不需要返回 | `throws -> T` |
| 异步回调（闭包） | `Result<T, JacoXxxError>` 或独立的 `success` / `failure` 闭包 |
| 失败不需要原因，只是"取不到" | 直接 `T?`（Optional），不用搞个 Error |

### throws 示例

```swift
/// 解析本地房间配置
/// - Throws: `JacoRoomError.invalidRoomId` 房间 ID 异常；网络失败抛 `.network(underlying:)`
func loadRoomConfig(lid: String) throws -> JacoRoomConfig {
    guard !lid.isEmpty else { throw JacoRoomError.invalidRoomId }
    // ...
}

// 调用
do {
    let config = try loadRoomConfig(lid: lid)
} catch let error as JacoRoomError {
    Logger_e(.live, "[直播] 加载配置失败 \(error.localizedDescription)")
    // 业务兜底
} catch {
    Logger_e(.live, "[直播] 未知错误 \(error)")
}
```

### 异步回调示例（与项目 BNet 风格一致）

项目网络层是 `success` / `failure` 双闭包，业务层封装时**保持一致**：

```swift
func enterRoom(lid: String,
               success: @escaping (JacoRoomModel) -> Void,
               failure: @escaping (JacoRoomError) -> Void) {
    guard !lid.isEmpty else {
        failure(.invalidRoomId)
        return
    }
    JacoRoomNetwork.enter(lid: lid).HTTPRequest { json in
        // ...
    } failure: { error in
        failure(.network(underlying: error))
    }
}
```

如果需要单个回调，用 `Result`：

```swift
func enterRoom(lid: String, completion: @escaping (Result<JacoRoomModel, JacoRoomError>) -> Void) { }
```

### Optional vs Error 选型

```swift
// ✅ 只是"找不到"，不需要原因
func user(by uid: String) -> JacoUserModel?

// ✅ 失败原因多样，需要错误信息
func login(account: String, password: String) throws -> JacoUserModel
```

## 三、不要做的事

- ❌ 用 `assert` / `fatalError` / `preconditionFailure` 替代错误处理（见 `best-practices.md`）
- ❌ 把所有 case 塞到一个 `JacoCommonError`，错误失去语义
- ❌ `throws` 函数里只 `throw NSError(domain: ...)`，类型 opaque 无法 switch
- ❌ Result 失败 case 用 `Error`（基类）而不是具体的 `JacoXxxError`，调用方拿到要类型转换
- ❌ catch 后什么都不做（`} catch { }`），错误必须 `Logger_e` 至少落一条

## 四、错误日志与用户提示分离

- **日志**：用 `Logger_e` 写完整的错误描述（含 underlying 详情、调用上下文），供问题排查
- **UI 提示**：用 `error.localizedDescription` 或更友好的文案展示给用户，不要直接 `print(error)` 弹 toast

```swift
} catch let error as JacoRoomError {
    Logger_e(.live, "[直播] 进房失败 lid:\(lid) \(error)")
    JacoToast.show(error.errorDescription ?? "进入房间失败，请稍后重试")
}
```
