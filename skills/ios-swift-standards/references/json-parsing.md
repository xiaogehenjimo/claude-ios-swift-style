# JSON 解析规范（HandyJSON）

> 引用自 `SKILL.md` 第四章。所有 Model 解析走 HandyJSON。

## Model 定义

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

## 反序列化

```swift
// 从字典
if let model = JacoUserModel.deserialize(from: jsonDict) {
    // 使用 model
}

// 从 JSON 字符串
if let model = JacoUserModel.deserialize(from: jsonString) { }

// 数组从字典数组
if let list = [JacoItemModel].deserialize(from: arrayOfDicts) {
    let validList = list.compactMap {  }
}

// 从网络回包（json 是 [String: Any]?）
JacoXxxNetwork.list(params: [:]).HTTPRequest { json in
    if let data = json?["data"] as? [String: Any],
       let model = JacoUserModel.deserialize(from: data) {
        self.model = model
    }
    // 或者 list
    if let array = json?["list"] as? [[String: Any]],
       let list = [JacoItemModel].deserialize(from: array) {
        self.dataList = list.compactMap {  }
    }
}
```

## 序列化

```swift
// 序列化为字典
let dict = model.toJSON()

// 序列化为 JSON 字符串
let jsonString = model.toJSONString()
```

## 规则

- Model 文件命名：`JacoXxxModel.swift`
- 属性必须给默认值（避免解析失败时 nil 崩溃）
- 嵌套 Model 同样继承 `BaseObjcModel`
