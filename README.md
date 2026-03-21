# claude-ios-swift-style

A Claude Code plugin providing iOS Swift code standards and templates.

## 包含内容

- 文件命名 & 头注释规范
- 日志打印（Logger_i / JacoSafeDebug）
- 网络请求（BNet / NetworkAPI）
- JSON 解析（HandyJSON）
- 布局约束（SnapKit）
- 颜色使用（UIColor.hex / ColorTheme）
- MVVM 模板（ViewController / ViewModel / Cell）
- 线程规范 & 其他最佳实践

## 安装（经验证的正确步骤）

在终端执行以下两条命令（缺一不可）：

```bash
# 第一步：注册 marketplace（会自动拉取 repo 到本地缓存）
claude plugin marketplace add xiaogehenjimo/claude-ios-swift-style

# 第二步：安装 skill
claude plugin install ios-swift-standards@claude-ios-swift-style
```

安装格式说明：`{skill目录名}@{marketplace名}`
- skill 目录名 = `skills/` 下的子目录名
- marketplace 名 = `.claude-plugin/marketplace.json` 中的顶层 `name` 字段

> ⚠️ 不要只修改 `settings.json`，必须执行 `marketplace add` 命令。

## 更新规范

```bash
# 1. 编辑 skills/ios-swift-standards/SKILL.md
# 2. 提交推送
git add skills/ios-swift-standards/SKILL.md
git commit -m "feat: 更新xxx规范"
git push

# 3. 所有已安装机器执行更新
claude plugin update ios-swift-standards@claude-ios-swift-style
```

## 目录结构

```
claude-ios-swift-style/
├── .claude-plugin/
│   ├── plugin.json          # 插件元数据
│   └── marketplace.json     # 插件列表声明（plugin install 依赖此文件）
├── skills/
│   └── ios-swift-standards/
│       └── SKILL.md         # 规范与模板主文件
└── README.md
```
