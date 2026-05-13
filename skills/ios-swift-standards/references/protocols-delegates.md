# 协议 / Delegate 模式

> 引用自 `SKILL.md`。命名规则见 `naming.md`；这里讲使用模式与陷阱。

## 一、Delegate 必须 weak（防循环引用）

```swift
// MARK: - 协议定义
protocol JacoRoomCellDelegate: AnyObject {     // ⚠ class-only，否则不能 weak
    func roomCell(_ cell: JacoRoomCell, didTapFollow user: JacoUserModel)
    func roomCellDidTapAvatar(_ cell: JacoRoomCell)
}

// MARK: - cell 持有
class JacoRoomCell: UITableViewCell {
    weak var delegate: JacoRoomCellDelegate?    // ⚠ 必须 weak
}

// MARK: - VC 实现
extension JacoRoomListViewController: JacoRoomCellDelegate {
    func roomCell(_ cell: JacoRoomCell, didTapFollow user: JacoUserModel) { }
    func roomCellDidTapAvatar(_ cell: JacoRoomCell) { }
}
```

**关键点**：
- 协议必须继承 `AnyObject`（或老写法 `: class`），否则属性不能 `weak`
- 持有 delegate 的属性必须 `weak var`，**没有任何例外**
- 不要在 deinit 里手动置 nil，weak 会自动处理

## 二、协议方法命名约定（参考 Apple）

第一个参数是发起者本身，方法名读起来像描述事件：

```swift
// ✅ 像 UITableViewDelegate 那样
func roomCell(_ cell: JacoRoomCell, didTapFollow user: JacoUserModel)
func roomCellDidTapAvatar(_ cell: JacoRoomCell)         // 无额外参数时

// ❌ 反人类语序
func didTapFollow(user: JacoUserModel, in cell: JacoRoomCell)
func tapAvatar()                                         // 不知道是谁
```

事件用语：`didXxx`（已发生）/ `willXxx`（即将）/ `shouldXxx`（询问能否）。

## 三、可选协议方法的两种写法

### Swift 推荐：protocol extension 默认实现

```swift
protocol JacoRoomCellDelegate: AnyObject {
    func roomCell(_ cell: JacoRoomCell, didTapFollow user: JacoUserModel)
    func roomCellDidTapAvatar(_ cell: JacoRoomCell)     // 想可选
}

// 默认空实现，实现方按需 override
extension JacoRoomCellDelegate {
    func roomCellDidTapAvatar(_ cell: JacoRoomCell) { }
}
```

### OC 互通才用：@objc optional

```swift
@objc protocol JacoOCBridgeDelegate {
    @objc optional func someOptionalMethod()
}
```

`@objc optional` 会丢失 Swift 类型推断，**只在需要被 OC 调用时**才用。

## 四、Delegate vs Closure：什么时候用哪个

| 场景 | 选 delegate | 选 closure |
|---|---|---|
| 事件 ≥ 3 个，业务相关 | ✅ | |
| 单一回调（"完成了"、"取消了"） | | ✅ |
| 1:1 关系（cell ↔ vc） | ✅ | ✅ |
| 1:N 通知（多处监听） | | NotificationCenter / Rx |
| 强类型，参数复杂 | ✅ | |
| 调用方实现成本要低 | | ✅ |

### Closure 替代示例

```swift
class JacoConfirmAlertView: UIView {
    var onConfirm: (() -> Void)?
    var onCancel: (() -> Void)?

    @objc private func confirmBtnDidTap() {
        onConfirm?()
    }
}

// 调用方
let alert = JacoConfirmAlertView()
alert.onConfirm = { [weak self] in
    guard let self else { return }
    self.handleConfirm()
}
```

**closure 同样要小心循环引用**：`[weak self]` 不是建议，是要求。

## 五、能力型协议命名 `-able` / `-ing`

不是 delegate 的纯能力协议，按 Swift 标准库风格命名：

```swift
protocol Reusable {
    static var reuseId: String { get }
}

protocol Logging {
    func log(_ message: String)
}

extension Reusable {
    static var reuseId: String { String(describing: self) }
}

extension UITableViewCell: Reusable { }
```

## 六、规则汇总

- delegate 协议必须 `: AnyObject`，持有方必须 `weak var`
- 协议方法首参为发起者本身（`func xxxView(_ view: XxxView, didDoSomething ...)`）
- 可选方法 → protocol extension 默认实现；不要乱用 `@objc optional`
- 单一回调用 closure，多事件用 delegate
- closure 内必须 `[weak self]`
- 能力协议用 `-able` / `-ing` 后缀
