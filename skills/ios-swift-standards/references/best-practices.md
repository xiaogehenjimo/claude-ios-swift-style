# 其他注意事项

> 引用自 `SKILL.md` 第十二章。零碎但重要的项目规则。

## 弱引用

```swift
// 闭包内必须捕获 weak self，避免循环引用
someTask { [weak self] result in
    guard let self else { return }
    self.updateUI()
}
```

## 安全访问

```swift
// 非空判断用 isNonEmpty（项目扩展）
if someString.isNonEmpty { }
if someArray.isNonEmpty { }
```

## 图片加载（Kingfisher）

```swift
// Swift 代码用 Kingfisher
imageView.kf.setImage(with: URL(string: urlString))
imageView.kf.setImage(with: URL(string: urlString), placeholder: UIImage(named: "placeholder"))

// OC 代码用 SDWebImage
```

## MMKV 持久化

```swift
// 读写用 JacoKVM（项目封装）
JacoKVM.someKey = value
let value = JacoKVM.someKey
```

## 数组取值

```swift
let list: [String] = ["xiaohong", "xiaoming"]
let name = list[safe: 0]
```

## 语法技巧

- **禁止使用强制解包**（`!`）。用 `if let` / `guard let` / `as?` / `[safe:]` 等安全方式。
- **禁止使用 `assert` / `assertionFailure` / `precondition` / `preconditionFailure` / `fatalError`**（受系统 API 约束的 `required init?(coder:)` 等场景除外）。Release 包里 `precondition` 会真崩，`assert` 在 Debug 也会让进程退出，掩盖问题且不可观测。改用：
  - 业务非法状态：`Logger_e` 打错误日志 + 安全 fallback（return / 兜底值）
  - 程序员错误：`Logger_e` + `JacoSafeDebug { debugPrint(...) }`，让 Debug 期能看到、Release 不崩
  - 不要用断言来"标记 TODO"，用真实代码兜底

```swift
// ❌ 禁止
assert(uid != 0, "uid 不能为 0")
guard let model = model else { fatalError("model 必须非空") }

// ✅ 正确
guard uid != 0 else {
    Logger_e(.common, "[Xxx] uid 异常为 0，跳过本次请求")
    return
}
guard let model = model else {
    Logger_e(.common, "[Xxx] model 为空")
    return
}
```

## 代码组织

### MARK 分组
常用分组（按出现顺序）：
```swift
// MARK: - Properties
// MARK: - UI
// MARK: - Lifecycle
// MARK: - Setup
// MARK: - Public
// MARK: - Private
// MARK: - Actions
// MARK: - Networking
```

### 一个文件只能有一个 class
- **单文件只允许一个主 class**（含 struct / enum 也按主类型计）。出现第二个独立 class 必须拆出新文件
- 同一类型的私有辅助 struct / enum 可放在同文件底部
- 拆分后命名：`JacoXxxXxx.swift`，遵循 `Jaco` 前缀规则

### 单文件过长 → 按职责拆 extension
**单文件代码量过多必须拆 extension**（经验阈值：超过 ~300 行或职责混杂就该拆），按职责切到独立文件：

| 用途 | 文件名 |
|---|---|
| UITableView/UICollectionView 代理 | `JacoXxxViewController+Delegate.swift` |
| 业务交互 / Action | `JacoXxxViewController+Action.swift` |
| 网络请求封装 | `JacoXxxViewController+Network.swift` |
| 通知 / KVO / Rx 绑定 | `JacoXxxViewController+Binding.swift` |

### 每个 extension 必须加 MARK 注释
**只要写了 extension，无论同文件还是拆出去，都必须在 extension 上方加 `// MARK: -` 说明该 extension 的用途**：

```swift
// MARK: - UITableViewDataSource
extension JacoRoomListViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int { ... }
}

// MARK: - UITableViewDelegate
extension JacoRoomListViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) { ... }
}

// MARK: - 网络请求
extension JacoRoomListViewController {
    func fetchRoomList() { ... }
}
```

规则：
- MARK 内容用中文或英文协议名，必须表达"这个 extension 负责什么"
- 严禁出现没有 MARK 的裸 extension
- 不允许多个 extension 共用一个 MARK
