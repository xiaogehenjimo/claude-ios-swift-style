---
name: ios-swift-standards
description: |
  iOS Swift 代码规范与模板。当用户要求编写 Swift 代码、创建新文件、实现功能模块、网络请求、UI 布局、数据解析时触发。
  包含：文件命名/头注释规范、日志打印、网络请求(BNet)、JSON解析(HandyJSON)、布局约束(SnapKit)、颜色使用、视图模板、MVVM架构模板。
  Trigger when: writing Swift code, creating Swift files, implementing iOS features, or when user says /ios-std or /swift-std
version: 1.1.0
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
---

# iOS Swift 代码规范 & 模板

> 本 skill 适用于所有 iOS Swift 项目。采用渐进式披露：本文件只放每次都要遵守的核心规则和导航表，详细规范和模板按需读取 `references/` 和 `templates/`。

---

## 一、文件命名（强制）

- 所有新建 Swift 文件必须以项目前缀开头（如 `Jaco`）
- 示例：`JacoUserViewController.swift`、`JacoUserCell.swift`、`JacoUserNetwork.swift`

## 二、文件头注释（强制）

```swift
//
//  JacoXxxXxx.swift
//  live
//
//  Created by <当前电脑登录用户名> on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  文件功能的一行中文描述
```

- 年份/日期使用创建当天（通过 `date +%Y/%-m/%-d` 动态获取）
- **author 必须动态获取当前电脑登录用户名**：创建 Swift 文件前先执行 `whoami` 拿到用户名，再写入 `Created by` 一行；不要硬编码任何用户名
- 最后一行用中文简述文件功能

---

## 三、按需查阅（渐进式披露）

下表列出所有详细规范和模板。**只在涉及对应主题时再 Read 对应文件**，不要一次性全部加载。

### references/ — 规范类

| 主题 | 何时读取 | 文件 |
|---|---|---|
| 命名规范 | 取变量/类型/枚举/协议/方法名时 | `references/naming.md` |
| 访问控制 | 标 `private` / `private(set)` / `final` / Pod 内 `public` 时 | `references/access-control.md` |
| 常量与魔法值 | 写硬编码数字/字符串 / 拆 Const 时 | `references/constants.md` |
| 日志打印 | 写 `Logger_i` / `JacoSafeDebug` / `print` 时 | `references/logging.md` |
| 网络请求 | 写 `JacoXxxNetwork` / `HTTPRequest` 时 | `references/network.md` |
| JSON 解析 | 写 Model / `deserialize` 时 | `references/json-parsing.md` |
| 错误处理 | 定义 Error 枚举 / throws / Result / 错误传递时 | `references/error-handling.md` |
| 协议 / Delegate | 定义协议 / weak delegate / closure vs delegate 选型 | `references/protocols-delegates.md` |
| SnapKit 布局 | 写 `snp.makeConstraints` 时 | `references/snapkit-layout.md` |
| 颜色使用 | 写 `UIColor` / `ColorTheme` 时 | `references/colors.md` |
| 视图属性 | 写 `lazy var` 子视图时 | `references/view-styling.md` |
| 线程切换 | 写 `DispatchQueue` / `JacoThread` 时 | `references/threading.md` |
| **函数注释** | **写函数 / 添加参数 / 对外暴露方法时（强制）** | `references/function-comments.md` |
| 其他实践 | 弱引用 / 图片加载 / MMKV / 数组安全取值 / 代码组织 / 禁用 assert | `references/best-practices.md` |

### templates/ — 代码模板

| 模板 | 何时读取 | 文件 |
|---|---|---|
| ViewController | 创建 `JacoXxxViewController.swift` 时 | `templates/view-controller.md` |
| ViewModel | 创建 `JacoXxxViewModel.swift` 时 | `templates/view-model.md` |
| TableView Cell | 创建 `JacoXxxCell.swift` 时 | `templates/tableview-cell.md` |

---

## 快速扩展指南

- 新增规范：在 `references/` 下新建 `xxx.md` 并在上表加一行
- 新增模板：在 `templates/` 下新建 `xxx.md` 并在上表加一行
- 修改本文件只在涉及核心规则（命名、头注释、整体结构）时；其他规则改对应子文件即可
