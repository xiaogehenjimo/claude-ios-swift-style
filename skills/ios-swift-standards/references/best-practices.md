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

不要使用强制解包（`!`），用 `if let` / `guard let` / `as?` / `[safe:]` 等安全方式。

## 代码组织

- `// MARK: - Lifecycle` / `// MARK: - Setup` / `// MARK: - Actions` 分组
- 大文件拆 extension 到单独文件：`JacoXxxViewController+Delegate.swift`
- 一个文件只包含一个主要类型
