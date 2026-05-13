# 常量与魔法值管理

> 引用自 `SKILL.md`。源码里散落的数字/字符串字面量是项目腐烂的最大来源，必须集中。

## 一、什么是魔法值

源码中没有语义、看不出来源的字面量：

```swift
// ❌ 全是魔法值
view.layer.cornerRadius = 8
titleLabel.snp.makeConstraints { .top.equalToSuperview().offset(24) }
let url = "https://api.jaco.live/v1/room/list.json"
if status == 3 { ... }                    // 3 是什么？
UserDefaults.standard.set(true, forKey: "has_shown_guide")
```

## 二、解决方案：按作用域集中

### 1. 文件内 / 单个类用：私有 enum 命名空间

页面专属、不外用的常量放文件底部的 `private enum Const`：

```swift
class JacoRoomListViewController: ViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        titleLabel.snp.makeConstraints {
            .top.equalToSuperview().offset(Const.topInset)
            .leading.trailing.equalToSuperview().inset(Const.hInset)
        }
        avatarImageView.layer.cornerRadius = Const.avatarCornerRadius
    }
}

// MARK: - Const
private extension JacoRoomListViewController {
    enum Const {
        static let topInset: CGFloat = 24
        static let hInset: CGFloat = 16
        static let avatarCornerRadius: CGFloat = 20
        static let pageSize: Int = 20
    }
}
```

为什么用 `enum` 不用 `struct`：enum 不能被实例化，纯命名空间，意图最清晰。

### 2. 模块内多文件共用：模块级常量类型

```swift
// JacoRoomConst.swift
enum JacoRoomConst {
    static let maxOnlineCount = 1000
    static let heartbeatInterval: TimeInterval = 30
    enum Layout {
        static let cellHeight: CGFloat = 88
        static let cardCornerRadius: CGFloat = 12
    }
}

// 调用
let height = JacoRoomConst.Layout.cellHeight
```

### 3. 全局共用：分类清晰的全局常量

| 类型 | 命名 | 用途 |
|---|---|---|
| URL / 接口路径 | 走 Network 枚举的 `path`（见 `network.md`） | 不再到处出现字符串 URL |
| 通知名 | `extension Notification.Name { static let xxx = ... }` | 类型安全的通知 |
| UserDefaults / MMKV key | `JacoKVM` 属性（见 `best-practices.md`） | 不裸写 forKey |
| Storyboard / xib 名 | `enum JacoStoryboard { static let main = "Main" }` | 防拼写错误 |

```swift
extension Notification.Name {
    static let jacoUserDidLogin = Notification.Name("jaco.user.didLogin")
    static let jacoRoomDidEnter = Notification.Name("jaco.room.didEnter")
}

NotificationCenter.default.post(name: .jacoUserDidLogin, object: nil)
```

## 三、什么情况下可以裸写字面量

少数情况允许，但必须**显而易见、无歧义**：

- `0` / `1` / `-1` 这种数学常量
- `isHidden = true` 这种语义自解释的布尔
- 单元测试里的固定输入数据
- `if list.isEmpty { return }` 这种边界判断

```swift
view.isHidden = true            // ✅ OK，语义清晰
btn.cornerRadius = 8            // ❌ 8 必须进 Const
```

## 四、规则

- 控件尺寸 / 间距 / 时长 / 字号 / 颜色 hex 等只要在 2 个地方以上出现，必须抽常量
- enum case 数字（`status == 3`）必须 → 替换为带语义的 enum case 或常量
- 字符串 URL 一律走 `JacoXxxNetwork`，源码中不允许出现裸 https 字符串
- 通知名、UserDefaults key、Storyboard 名 → 上述集中处
- 文件内常量统一放文件底部 `private extension Xxx { enum Const { ... } }`，不要散在类的顶部
