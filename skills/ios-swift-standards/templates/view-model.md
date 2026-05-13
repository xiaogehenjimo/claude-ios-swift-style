# ViewModel 模板

> 引用自 `SKILL.md` 第九章。MVVM 架构下的 ViewModel 标准结构。

```swift
//
//  JacoXxxViewModel.swift
//  live
//
//  Created by <当前电脑登录用户名> on 2026/3/21.
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
                self.dataList = list.compactMap {  }
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

## 使用要点

- 头注释 `Created by` 后用 `whoami` 拿到的用户名替换 `<当前电脑登录用户名>`
- 业务路径用 `Logger_i` 打点，错误用 `Logger_e`
- `loadData()` 必须有 `isLoading` 重入保护
