# TableView Cell 模板

> 引用自 `SKILL.md` 第十章。列表 Cell 的标准结构。

```swift
//
//  JacoXxxCell.swift
//  live
//
//  Created by <当前电脑登录用户名> on 2026/3/21.
//  Copyright © 2026 weo. All rights reserved.
//  Xxx 列表 Cell

import UIKit
import Kingfisher

class JacoXxxCell: UITableViewCell {

    static let reuseId = "JacoXxxCell"

    // MARK: - UI
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

    // MARK: - Init
    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        selectionStyle = .none
        configUI()
    }

    required init?(coder: NSCoder) { fatalError() }

    // MARK: - Layout
    private func configUI() {
        contentView.addSubview(avatarImageView)
        contentView.addSubview(titleLabel)

        avatarImageView.snp.makeConstraints {
            .leading.equalToSuperview().offset(16)
            .centerY.equalToSuperview()
            .size.equalTo(40)
        }
        titleLabel.snp.makeConstraints {
            .leading.equalTo(avatarImageView.snp.trailing).offset(12)
            .trailing.equalToSuperview().offset(-16)
            .centerY.equalToSuperview()
        }
    }

    // MARK: - Configure
    func configure(with model: JacoXxxModel) {
        titleLabel.text = model.name
        avatarImageView.kf.setImage(with: URL(string: model.avatar))
    }
}
```

## 使用要点

- 头注释 `Created by` 后用 `whoami` 拿到的用户名替换 `<当前电脑登录用户名>`
- 文件命名 `JacoXxxCell.swift`，`reuseId` 静态属性必须有
- 图片用 Kingfisher，不要在 cell 里直接处理网络
