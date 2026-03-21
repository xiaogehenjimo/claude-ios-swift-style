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

## 安装

在 Claude Code 中运行：

```
/install-plugin xiaogehenjimo/claude-ios-swift-style
```

或在 `~/.claude/settings.json` 中添加：

```json
{
  "extraKnownMarketplaces": {
    "claude-ios-swift-style": {
      "source": {
        "source": "github",
        "repo": "xiaogehenjimo/claude-ios-swift-style"
      }
    }
  }
}
```

然后运行 `/install-plugin claude-ios-swift-style@ios-swift-standards`。

## 更新规范

直接编辑 `skills/ios-swift-standards/SKILL.md`，提交 push 后，在 Claude Code 中运行：

```
/update-plugins
```

## 目录结构

```
claude-ios-swift-style/
├── .claude-plugin/
│   └── plugin.json          # 插件元数据
├── skills/
│   └── ios-swift-standards/
│       └── SKILL.md         # 规范与模板主文件
└── README.md
```
