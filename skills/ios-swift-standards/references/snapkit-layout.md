# 布局约束规范（SnapKit）

> 引用自 `SKILL.md` 第五章。所有约束走 SnapKit，不要混用 AutoresizingMask。

## 基础用法

```swift
// 添加子视图后再设约束
view.addSubview(titleLabel)
view.addSubview(imageView)
view.addSubview(btnContainer)

titleLabel.snp.makeConstraints {
    .top.equalToSuperview().offset(16)
    .leading.trailing.equalToSuperview().inset(24)
}

imageView.snp.makeConstraints {
    .top.equalTo(titleLabel.snp.bottom).offset(12)
    .centerX.equalToSuperview()
    .size.equalTo(CGSize(width: 80, height: 80))
}
```

## 更新 / 重置约束

```swift
// 更新已存在的约束（不能改 relation/attribute）
titleLabel.snp.updateConstraints {
    .top.equalToSuperview().offset(isExpanded ? 24 : 16)
}

// 重新设置所有约束
imageView.snp.remakeConstraints {
    .center.equalToSuperview()
    .size.equalTo(100)
}
```

## 相对布局

```swift
// 相对于某个视图
.top.equalTo(headerView.snp.bottom).offset(8)
.leading.equalTo(iconView.snp.trailing).offset(12)

// 等宽/等高
.width.equalTo(otherView)
.width.equalTo(otherView).multipliedBy(0.5)

// 优先级
.width.lessThanOrEqualTo(200).priority(.high)
```

## 规则

- 优先使用闭包简写（项目主流风格）
- `make` 写法也可，保持文件内一致即可
- 不要混用 AutoresizingMask 和 SnapKit
- 设约束前确保 `translatesAutoresizingMaskIntoConstraints` 已被 SnapKit 自动设为 false
