# 网络请求规范（BNet）

> 引用自 `SKILL.md` 第三章。所有 HTTP 接口走 BNet 封装。

## 定义 Network 枚举

```swift
import BNet

enum JacoXxxNetwork {
    /// 获取列表
    case list(params: [String: Any])
    /// 详情
    case detail(id: String)
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
        case .detail:  return "/bff/1/xxx/detail.json"
        case .submit:  return "/bff/1/xxx/submit.json"
        }
    }

    var method: APIMethod {
        switch self {
        case .list:    return .get
        case .detail:  return .get
        case .submit:  return .post
        }
    }

    var parameters: APIParameters? {
        switch self {
        case .list(let params):   return params
        case .detail(let id):     return ["id": id]
        case .submit(let params): return params
        }
    }
}
```

## 发起请求

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

## 规则

- Network 文件命名：`JacoXxxNetwork.swift`
- 每个接口写一个 case，case 名称用英文动词/名词
- 失败回调不能省略（哪怕是空的 `{ _ in }`）
- UI 操作必须在主线程，用 `HTTPRequest` 或在 `HTTPRequestInChildThread` 内手动 dispatch
