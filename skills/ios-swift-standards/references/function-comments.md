# 函数注释规范

> 引用自 `SKILL.md`。所有「关键函数」必须写完整 DocC 注释，参数说明用中文。

## 一、必须写注释的函数（关键函数）

满足以下任一条件就必须写完整注释：

- `public` / `open` / `internal`（默认）作用域、对外暴露的函数
- 协议方法、`@objc` 暴露的方法
- ViewModel / Service / Manager 类的对外业务方法
- 网络请求封装、数据转换、复杂业务逻辑
- 闭包参数 ≥ 2 个，或参数语义不直观的函数
- 静态工厂方法 / 单例访问入口

## 二、可以省略注释的函数

- `private` 的简单 setter / getter
- 系统生命周期 override（`viewDidLoad` / `viewWillAppear` 等，除非内部有特殊逻辑）
- 一行表达式的 computed property
- 简单的 `@IBAction`（但如果触发了复杂业务逻辑还是要写）

## 三、注释格式（Swift DocC）

### 多参数 — 用 `Parameters:` 块

```swift
/// 加载指定页码的房间列表数据
///
/// 内部会做重入保护，已在 loading 中时会直接 return。
/// 成功后通过 dataSubject 推送，失败时通过 errorSubject 推送错误信息。
///
/// - Parameters:
///   - page: 页码，从 1 开始，默认 1
///   - pageSize: 每页条数，范围 10 ~ 50，超出会被截断
///   - keyword: 搜索关键字，传 nil 表示不过滤
/// - Returns: 是否成功发起请求（重入时返回 false）
func loadRoomList(page: Int = 1,
                  pageSize: Int,
                  keyword: String?) -> Bool {
    // ...
}
```

### 单参数 — 用 `Parameter` 简写

```swift
/// 根据房间 ID 进入直播间
/// - Parameter lid: 直播间唯一 ID，不能为空字符串
func enterRoom(lid: String) {
    // ...
}
```

### 带闭包参数 — 闭包要单独说明触发时机

```swift
/// 发起支付请求
///
/// - Parameters:
///   - orderId: 订单 ID，由服务端 /order/create 接口下发
///   - amount: 支付金额，单位为分（如 100 表示 1 元）
///   - completion: 支付完成回调，主线程触发；success 表示是否支付成功，error 仅在失败时非空
func requestPay(orderId: String,
                amount: Int,
                completion: @escaping (_ success: Bool, _ error: Error?) -> Void) {
    // ...
}
```

### 抛出异常 — 必须写 `Throws`

```swift
/// 解析本地配置文件
/// - Parameter path: 配置文件绝对路径
/// - Returns: 解析后的配置 Model
/// - Throws: `JacoConfigError.fileNotFound` 文件不存在；`JacoConfigError.invalidFormat` 格式错误
func parseConfig(at path: String) throws -> JacoConfigModel {
    // ...
}
```

## 四、参数注释内容要点

每个参数注释必须覆盖以下要素（按需取舍）：

| 要素 | 示例 |
|---|---|
| 业务含义 | "直播间唯一 ID" 而不是 "the room id" |
| 取值范围 / 约束 | "范围 10 ~ 50"、"非空字符串"、"从 1 开始" |
| 默认值 / nil 含义 | "传 nil 表示不过滤"、"默认 1" |
| 单位 | "单位为分"、"单位毫秒" |
| 来源 / 上下文 | "由服务端 /order/create 接口下发" |
| 触发线程（闭包） | "主线程触发"、"子线程回调" |

❌ 不合格示例：
```swift
/// - Parameter id: id    // 等于没写
/// - Parameter url: 链接  // 没说什么链接、能否为空
```

✅ 合格示例：
```swift
/// - Parameter id: 主播 uid，来自登录态 JacoKVM.uid，不能为 0
/// - Parameter url: 直播流 m3u8 拉流地址，必须以 https 开头
```

## 五、文件内函数排版

```swift
class JacoXxxViewModel {

    // MARK: - Public API

    /// 启动定时刷新，每 30s 拉一次最新房间列表
    func startAutoRefresh() { ... }

    /// 停止定时刷新，释放定时器
    func stopAutoRefresh() { ... }

    // MARK: - Private

    private func setupTimer() { ... }   // private 简单方法可不写
}
```

## 六、规则汇总

- 关键函数（见第一节）**必须**写 DocC 注释，不允许只写一行 `// xxx` 应付
- 参数注释**必须中文**，覆盖业务含义、范围、默认值/nil 含义、单位等
- 闭包参数必须说明触发线程
- `Throws` 必须列出可能抛出的具体错误类型
- private 简单方法可省略，但内部逻辑复杂时仍建议写
