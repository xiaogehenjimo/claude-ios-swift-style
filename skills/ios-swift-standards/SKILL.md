---
name: ios-swift-standards
description: |
  iOS Swift 代码规范与模板。当用户要求编写 Swift 代码、创建新文件、实现功能模块、网络请求、UI 布局、数据解析时触发。
  包含：文件命名/头注释规范、日志打印、网络请求(BNet)、JSON解析(HandyJSON)、布局约束(SnapKit)、颜色使用、视图模板、MVVM架构模板。
  Trigger when: writing Swift code, creating Swift files, implementing iOS features, or when user says /ios-std or /swift-std
version: 1.0.1
user-invocable: true
allowed-tools: "Read, Write, Edit, Bash, Glob, Grep"
---

# iOS Swift 代码规范 & 模板

> 本 skill 适用于所有 iOS Swift 项目。包含代码规范、常用模板和注意事项。

---

## 一、文件规范

### 1.1 文件命名
- 所有新建 Swift 文件必须以项目前缀开头（如 `Jaco`）
- 示例：`JacoUserViewController.swift`、`JacoUserCell.swift`、`JacoUserNetwork.swift`

### 1.2 文件头注释（必须）
```swift
//
//  JacoXxxXxx.swift
//  live
//
//  Created by xuqinqiang on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  文件功能的一行中文描述
```
- 年份使用创建当年
- author 固定为 `xuqinqiang`
- 最后一行用中文简述文件功能

---

## 二、日志打印规范

### 2.1 业务日志（持久化，生产可查）
```swift
// info 级别 — 常规业务流程
Logger_i(.home, "首页请求发起 isFirstLoad:\(isFirstLoad)")
Logger_i(.network, "[网络] 状态变化 \(status)")
Logger_i(.live, "[直播] 进入房间 lid:\(lid)")
Logger_i(.common, "通用业务事件")
Logger_i(.homeStream, "[流] 开始播放")

// error 级别 — 错误、异常
Logger_e(.network, "[网络] 请求失败 error:\(error)")

// warning 级别 — 警告
Logger_w(.common, "非预期状态")
```

### 2.2 调试日志（仅 Debug/Profile 生效，Release 自动关闭）
```swift
// 单行调试日志优先使用，Release 下是 no-op
JacoSafeDebugLog("调试信息")

// 多行代码情况下用 JacoSafeDebug 包裹
JacoSafeDebug {
    debugPrint("调试信息：\(someValue)")
}
```

### 规则
- **不要**在正式代码中裸用 `print()`，统一使用 `Logger_i` / `JacoSafeDebug` / `JacoSafeDebugLog` 
- 日志 category 选最匹配的：`.common` / `.home` / `.network` / `.live` / `.homeStream`
- 关键路径必须打 `Logger_i`，调试辅助信息用 `JacoSafeDebug` / `JacoSafeDebugLog` 

---

## 三、网络请求规范（BNet）

### 3.1 定义 Network 枚举
```swift
import BNet

enum JacoXxxNetwork {
    /// 获取列表
    case list(params: [String: Any])
    /// 提交
    case submit(params: [String: Any])
}

extension JacoXxxNetwork: NetworkAPI {

    var ip: APIHost {
        return NetworkConfig.baseURL
    }

    var path: APIPath {
        switch self {
        case .list:    return "/bff/1/xxx/list.json"
        case .submit:  return "/bff/1/xxx/submit.json"
        }
    }

    var method: APIMethod {
        switch self {
        case .list:    return .get
        case .submit:  return .post
        }
    }

    var parameters: APIParameters? {
        switch self {
        case .list(let params):   return params
        case .submit(let params): return params
        }
    }
}
```

### 3.2 发起请求
```swift
// 主线程回调（UI 相关，有 loading）
JacoXxxNetwork.list(params: ["page": 1]).HTTPRequest { [weak self] json in
    guard let self else { return }
    // json: [String: Any]?
    Logger_i(.common, "列表请求成功")
} failure: { error in
    Logger_e(.common, "列表请求失败 \(error)")
}

// 子线程回调（后台处理，无 loading）
JacoXxxNetwork.submit(params: params).HTTPRequestInChildThread { json in
    // 注意：此处在子线程，操作 UI 需 DispatchQueue.main.async
} failure: { error in }

// 带 loading 插件
JacoXxxNetwork.detail(id: "123").HTTPRequest(plugins: [NetworkLoadingPlugin()]) { json in
    // ...
} failure: { _ in }
```

### 规则
- Network 文件命名：`JacoXxxNetwork.swift`
- 每个接口写一个 case，case 名称用英文动词/名词
- 失败回调不能省略（哪怕是空的 `{ _ in }`）
- UI 操作必须在主线程，用 `HTTPRequest` 或在 `HTTPRequestInChildThread` 内手动 dispatch

---

## 四、JSON 解析规范（HandyJSON）

### 4.1 Model 定义
```swift
// 继承 HandyJSONModel（或项目中的 BaseModel）
class JacoUserModel: BaseObjcModel {
    var uid: String = ""
    var name: String = ""
    var avatar: String = ""
    var level: Int = 0
    var list: [JacoItemModel] = []

    required init() {}
}
```

### 4.2 反序列化
```swift
// 从字典
if let model = JacoUserModel.deserialize(from: jsonDict) { }

// 从 JSON 字符串
if let model = JacoUserModel.deserialize(from: jsonString) { }

// 数组从字典数组
if let list = [JacoItemModel].deserialize(from: arrayOfDicts) {
    let validList = list.compactMap { $0 }
}

// 从网络回包（json 是 [String: Any]?）
JacoXxxNetwork.list(params: [:]).HTTPRequest { json in
    if let data = json?["data"] as? [String: Any],
       let model = JacoUserModel.deserialize(from: data) {
        self.model = model
    }
    if let array = json?["list"] as? [[String: Any]],
       let list = [JacoItemModel].deserialize(from: array) {
        self.dataList = list.compactMap { $0 }
    }
}
```

### 4.3 序列化
```swift
let dict = model.toJSON()
let jsonString = model.toJSONString()
```

### 规则
- Model 文件命名：`JacoXxxModel.swift`
- 属性必须给默认值（避免解析失败时 nil 崩溃）
- 嵌套 Model 同样继承 `BaseObjcModel`

---

## 五、布局约束规范（SnapKit）

### 5.1 基础用法
```swift
view.addSubview(titleLabel)
view.addSubview(imageView)

titleLabel.snp.makeConstraints {
    $0.top.equalToSuperview().offset(16)
    $0.leading.trailing.equalToSuperview().inset(24)
}

imageView.snp.makeConstraints {
    $0.top.equalTo(titleLabel.snp.bottom).offset(12)
    $0.centerX.equalToSuperview()
    $0.size.equalTo(CGSize(width: 80, height: 80))
}
```

### 5.2 更新 / 重置约束
```swift
// 更新已存在的约束
titleLabel.snp.updateConstraints {
    $0.top.equalToSuperview().offset(isExpanded ? 24 : 16)
}

// 重新设置所有约束
imageView.snp.remakeConstraints {
    $0.center.equalToSuperview()
    $0.size.equalTo(100)
}
```

### 5.3 相对布局
```swift
$0.top.equalTo(headerView.snp.bottom).offset(8)
$0.leading.equalTo(iconView.snp.trailing).offset(12)
$0.width.equalTo(otherView).multipliedBy(0.5)
$0.width.lessThanOrEqualTo(200).priority(.high)
```

### 规则
- 优先使用 `$0` 简写（项目主流风格）
- 不要混用 AutoresizingMask 和 SnapKit
- `addSubview` 和约束统一写在 `configUI()` 函数中

---

## 六、颜色使用规范

### 6.1 动态主题色（推荐）
```swift
// 跟随深/浅色主题自动切换（SwiftTheme）
view.theme_backgroundColor = ColorTheme.colorFFF_181818
label.theme_textColor = ColorTheme.color333333
btn.layer.theme_borderColor = ColorTheme.cgColor_bw_10
```

### 6.2 固定颜色 hex
```swift
UIColor.hex("8710FF")
UIColor.hex("515151").withAlphaComponent(0.5)
UIColor.hex("#FFFFFF")
```

### 6.3 业务语义色
```swift
LiveRoomColor.color_8710FF      // 主题紫
LiveRoomColor.color_000000_40   // 半透明黑
ColorLight.color333             // 仅浅色
ColorDark.color000              // 仅深色
```

### 规则
- 跟主题切换的用 `ColorTheme.xxx`（theme_ 系列属性）
- 固定颜色用 `UIColor.hex("xxxxxx")`，不用 `UIColor(red:green:blue:alpha:)`
- 业务语义色优先从 `ColorTheme` / `LiveRoomColor` 取

---

## 七、视图定义规范（lazy 属性）

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
    return btn
}()

```

### 规则
- 所有子视图用 `private lazy var`
- 控件初始化逻辑写在闭包内，不要散落在 `viewDidLoad`

---

## 八、ViewController 模板

```swift
//
//  JacoXxxViewController.swift
//  live
//
//  Created by xuqinqiang on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  Xxx 页面

import UIKit
import SnapKit
import RxSwift

class JacoXxxViewController: ViewController {

    // MARK: - Properties
    private let disposeBag = DisposeBag()
    private lazy var viewModel = JacoXxxViewModel()

    // MARK: - UI
    private lazy var titleLabel: UILabel = {
        let label = UILabel()
        label.theme_textColor = ColorTheme.color333333
        label.font = .ubuntu(18, weight: .bold)
        return label
    }()

    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
        configUI()
        bindViewModel()
        loadData()
    }

    deinit {
        JacoSafeDebug { debugPrint("JacoXxxViewController dealloc") }
    }

    // MARK: - Setup
    private func configUI() {
        view.addSubview(titleLabel)
        titleLabel.snp.makeConstraints {
            $0.top.equalTo(view.safeAreaLayoutGuide).offset(16)
            $0.leading.trailing.equalToSuperview().inset(24)
        }
    }

    private func bindViewModel() {
        viewModel.dataSubject
            .observe(on: MainScheduler.instance)
            .subscribe(onNext: { [weak self] data in
                guard let self else { return }
                // 更新 UI
            })
            .disposed(by: disposeBag)
    }

    private func loadData() {
        viewModel.loadData()
    }
}
```

---

## 九、ViewModel 模板

```swift
//
//  JacoXxxViewModel.swift
//  live
//
//  Created by xuqinqiang on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  Xxx 页面 ViewModel

import Foundation
import RxSwift
import BNet

class JacoXxxViewModel: NSObject {

    // MARK: - Output
    let dataSubject = PublishSubject<[JacoXxxModel]>()
    let errorSubject = PublishSubject<String>()

    // MARK: - State
    private(set) var dataList: [JacoXxxModel] = []
    private(set) var isLoading: Bool = false
    private let disposeBag = DisposeBag()

    // MARK: - Load
    func loadData() {
        guard !isLoading else { return }
        isLoading = true
        Logger_i(.common, "[Xxx] 开始请求数据")

        JacoXxxNetwork.list(params: [:]).HTTPRequest { [weak self] json in
            guard let self else { return }
            self.isLoading = false
            if let array = json?["list"] as? [[String: Any]],
               let list = [JacoXxxModel].deserialize(from: array) {
                self.dataList = list.compactMap { $0 }
                Logger_i(.common, "[Xxx] 请求成功 count:\(self.dataList.count)")
                self.dataSubject.onNext(self.dataList)
            }
        } failure: { [weak self] error in
            self?.isLoading = false
            Logger_e(.common, "[Xxx] 请求失败 \(error)")
            self?.errorSubject.onNext(error.localizedDescription)
        }
    }
}
```

---

## 十、TableView Cell 模板

```swift
//
//  JacoXxxCell.swift
//  live
//
//  Created by xuqinqiang on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  Xxx 列表 Cell

import UIKit
import Kingfisher

class JacoXxxCell: UITableViewCell {

    static let reuseId = "JacoXxxCell"

    private lazy var avatarImageView: UIImageView = {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFill
        iv.clipsToBounds = true
        iv.layer.cornerRadius = 20
        return iv
    }()

    private lazy var titleLabel: UILabel = {
        let label = UILabel()
        label.theme_textColor = ColorTheme.color333333
        label.font = .ubuntu(15, weight: .medium)
        return label
    }()

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        selectionStyle = .none
        configUI()
    }

    required init?(coder: NSCoder) { fatalError() }

    private func configUI() {
        contentView.addSubview(avatarImageView)
        contentView.addSubview(titleLabel)

        avatarImageView.snp.makeConstraints {
            $0.leading.equalToSuperview().offset(16)
            $0.centerY.equalToSuperview()
            $0.size.equalTo(40)
        }
        titleLabel.snp.makeConstraints {
            $0.leading.equalTo(avatarImageView.snp.trailing).offset(12)
            $0.trailing.equalToSuperview().offset(-16)
            $0.centerY.equalToSuperview()
        }
    }

    func configure(with model: JacoXxxModel) {
        titleLabel.text = model.name
        avatarImageView.kf.setImage(with: URL(string: model.avatar))
    }
}
```

---

## 十一、线程使用规范

```swift
// Jaco 封装
JacoThread.runOnGlobalQueue {
    // 后台处理
    JacoThread.runOnMain {
        // 回到主线程更新 UI
    }
}

// 标准 GCD
DispatchQueue.global().async {
    DispatchQueue.main.async { }
}
```

### 规则
- UI 操作必须在主线程
- 避免 `DispatchQueue.main.sync`（死锁风险）

---

## 十二、其他注意事项

### 弱引用
```swift
someTask { [weak self] result in
    guard let self else { return }
    self.updateUI()
}
```

### 非空判断
```swift
if someString.isNonEmpty { }
if someArray.isNonEmpty { }
```

### 图片加载
```swift
// Swift → Kingfisher
imageView.kf.setImage(with: URL(string: urlString))
// OC → SDWebImage
```

### MMKV 持久化
```swift
JacoKVM.someKey = value
let value = JacoKVM.someKey
```

### 数组取值
```swift
let list: [String] = ["xiaohong", "xiaoming"]
let name = list[safe: 0]
```

### 语法技巧
不要使用强制解包

### 代码组织
- `// MARK: - Lifecycle` / `// MARK: - Setup` / `// MARK: - Actions` 分组
- 大文件拆 extension：`JacoXxxViewController+Delegate.swift`
- 一个文件只包含一个主要类型

---

## 扩展指南

克隆本 repo 后在 `skills/ios-swift-standards/` 下可新增：
- `templates/` — 更多代码片段
- `patterns/` — 特定业务模式（直播、支付、RTC等）

修改 `SKILL.md` 后提交到 GitHub，其他机器 `/update-plugins` 即可同步。
