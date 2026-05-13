# ViewController 模板

> 引用自 `SKILL.md` 第八章。创建页面级控制器时使用此模板。

```swift
//
//  JacoXxxViewController.swift
//  live
//
//  Created by <当前电脑登录用户名> on 2026/3/21.
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
            .top.equalTo(view.safeAreaLayoutGuide).offset(16)
            .leading.trailing.equalToSuperview().inset(24)
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

## 使用要点

- 头注释 `Created by` 后用 `whoami` 拿到的用户名替换 `<当前电脑登录用户名>`
- 年份/日期用 `date +%Y/%-m/%-d` 动态获取
- 文件命名遵循 `JacoXxxViewController.swift` 前缀规则
