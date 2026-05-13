# 视图定义规范

> 引用自 `SKILL.md` 第七章。所有子视图统一用 `private lazy var` 定义。

## lazy 属性模板

```swift
// UILabel
private lazy var titleLabel: UILabel = {
    let label = UILabel()
    label.theme_textColor = ColorTheme.color333333
    label.font = UIFont.ubuntu(16, weight: .regular)
    label.textAlignment = .left
    label.numberOfLines = 0
    return label
}()

// UIButton
private lazy var confirmBtn: UIButton = {
    let btn = UIButton(type: .custom)
    btn.backgroundColor = UIColor.hex("8710FF")
    btn.layer.cornerRadius = 8
    btn.titleLabel?.font = .ubuntu(16, weight: .bold)
    btn.theme_setTitleColor(ColorTheme.colorFFF_FFF, forState: .normal)
    btn.setTitle("确认", for: .normal)
    return btn
}()
```

## 规则

- 所有子视图用 `private lazy var`
- 控件初始化逻辑写在闭包内，不要散落在 `viewDidLoad`
- `addSubview` 和 `snp.makeConstraints` 集中在 `configUI()` 函数中
