# 命名规范

> 引用自 `SKILL.md`。文件命名见 SKILL.md 一节，这里只讲文件内部的标识符。

## 一、类型（class / struct / enum / protocol）

- **大驼峰**，项目前缀 `Jaco`
- protocol 后缀：纯协议 `Xxxable` / `Xxxing`；delegate/datasource 类型协议 `JacoXxxDelegate` / `JacoXxxDataSource`
- enum 用单数名词：`enum JacoNetworkStatus`（不是 `Statuses`）

```swift
class JacoRoomListViewController { }
struct JacoUserModel { }
enum JacoNetworkStatus { }

protocol Reusable { }                      // 能力
protocol JacoRoomCellDelegate: AnyObject { } // delegate
```

## 二、变量 / 属性

- **小驼峰**
- 布尔变量必须用语义前缀：`is` / `has` / `should` / `can` / `did` / `will`
- 数组 / 集合用复数：`users`、`rooms`、`messages`
- 缩写遵循 Apple 风格：`url`、`id` 全小写；`URLs`、`IDs` 复数时连续大写

```swift
var userName: String = ""
var isLoading: Bool = false       // ✅
var loading: Bool = false         // ❌ 看不出是布尔
var hasNewMessage: Bool = false
var shouldShowAlert: Bool = false
var dataList: [JacoUserModel] = []   // 集合用复数

let id: String                    // ✅
let URLString: String             // ❌ 用 urlString
```

## 三、常量 / 类型属性

- 类型内常量：`static let xxx`，**小驼峰**（项目主流，不用大写下划线）
- 字符串 key / 通知名等可用 enum / struct 命名空间包起来（参考 `constants.md`）

```swift
class JacoRoomCell: UITableViewCell {
    static let reuseId = "JacoRoomCell"       // ✅ 小驼峰
    static let avatarSize: CGFloat = 40
}
```

## 四、enum case

- **小驼峰**
- 状态类 enum case 用动词或形容词：`.loading` / `.loaded` / `.failed(Error)`
- 关联值优先使用具名参数提升可读性

```swift
enum JacoLoadingState {
    case idle
    case loading
    case loaded(items: [JacoItemModel])     // 具名关联值
    case failed(error: Error)
}
```

## 五、函数 / 方法

- **小驼峰**，动词开头
- 参数标签让调用点像自然语言：`enterRoom(lid:)`、`fetchUser(by:)`
- 不要前缀 `get`（属性式 getter 直接用名词）；动作类用 `fetch` / `load` / `update`

```swift
func enterRoom(lid: String) { }              // ✅
func getRoom(_ lid: String) -> JacoRoom { }  // ❌ 用 fetchRoom

// 调用点应该读起来通顺
viewModel.fetchUser(by: uid)
viewModel.loadMore(after: lastId)
```

## 六、闭包 / typealias

- 复杂闭包用 typealias，命名 `XxxHandler` / `XxxCompletion` / `XxxBlock`
- 命名空间在类型前缀：`JacoXxx.XxxHandler`

```swift
typealias JacoPayCompletion = (_ success: Bool, _ error: Error?) -> Void

func requestPay(orderId: String, completion: @escaping JacoPayCompletion) { }
```

## 七、IBOutlet / IBAction

- IBOutlet：名词 + 控件类型，`titleLabel` / `confirmBtn`（项目里 button 习惯缩写为 `Btn`）
- IBAction：`xxxBtnDidTap` / `xxxBtnDidClick`，避免 `tapBtn`、`btnClick` 之类反人类语序

```swift
@IBOutlet private weak var titleLabel: UILabel!
@IBOutlet private weak var confirmBtn: UIButton!

@IBAction private func confirmBtnDidTap(_ sender: UIButton) { }
```

## 八、常见反例

| ❌ 反例 | ✅ 正确 | 原因 |
|---|---|---|
| `var flag: Bool` | `var isEnabled: Bool` | 布尔需语义前缀 |
| `var data: [User]` | `var users: [User]` | 集合用复数 |
| `func getUser() -> User` | `var user: User` 或 `func fetchUser()` | get 多余 |
| `class roomVC` | `class JacoRoomViewController` | 类型大驼峰 + 前缀 |
| `case Loading` | `case loading` | case 小驼峰 |
| `kReuseID = "xxx"` | `static let reuseId = "xxx"` | 项目用小驼峰，不留 OC 风格 `k` 前缀 |
