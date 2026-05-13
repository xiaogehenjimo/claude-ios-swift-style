# 颜色使用规范

> 引用自 `SKILL.md` 第六章。颜色按场景选择，不要散落 hex 字面量。

## 动态主题色（推荐）

```swift
// 跟随深/浅色主题自动切换（SwiftTheme）
view.theme_backgroundColor = ColorTheme.colorFFF_181818   // 白天白/夜间深色
label.theme_textColor = ColorTheme.color333333
btn.layer.theme_borderColor = ColorTheme.cgColor_bw_10
```

## 固定颜色 hex

```swift
// 固定不跟主题变化的颜色
UIColor.hex("8710FF")
UIColor.hex("515151").withAlphaComponent(0.5)
UIColor.hex("#FFFFFF")  // 带 # 也可以
```

## 业务语义色

```swift
// 直播间颜色
LiveRoomColor.color_8710FF    // 主题紫
LiveRoomColor.color_000000_40 // 半透明黑

// 明/暗
ColorLight.color333   // 仅浅色
ColorDark.color000    // 仅深色
```

## 规则

- 跟主题切换的用 `ColorTheme.xxx`（`theme_` 系列属性）
- 固定颜色用 `UIColor.hex("xxxxxx")`，不要用 `UIColor(red:green:blue:alpha:)` 写死
- 业务语义色优先从 `ColorTheme` / `LiveRoomColor` 取，避免散落的 hex 字面量
