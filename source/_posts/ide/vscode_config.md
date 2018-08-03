---
title: VSCODE常用配置
date: 2018-08-03 14:59:25
categories: 编程
tags: [VSCODE,编辑器,IDE]
---
vscode 常用配置
<!--more-->

扩展
```
Auto Close Tag
Auto Rename Tag
Chinese
EditorConfig for VS Code
ESLint
FreeMarker
Git Blame
Git History
GitLens
language-stylus
Markdown All in One
npm
npm Intellisense
Prettier
Reactjs code snippets
Sass
Sass Formatter
Sass Lint
Simple React Snippets
Typings auto installer
Vetur
vscode-icons
vscode-random
vscode-styled-components


```


设置
```
{
    "window.zoomLevel": 0,
    "editor.tabSize": 2,
    "emmet.triggerExpansionOnTab": true,
    "files.autoSave": "off",
    "javascript.implicitProjectConfig.experimentalDecorators": true,
    "workbench.iconTheme": "vscode-icons",
    "gitlens.advanced.messages": {
        "suppressCommitHasNoPreviousCommitWarning": false,
        "suppressCommitNotFoundWarning": false,
        "suppressFileNotUnderSourceControlWarning": false,
        "suppressGitVersionWarning": false,
        "suppressLineUncommittedWarning": false,
        "suppressNoRepositoryWarning": false,
        "suppressResultsExplorerNotice": false,
        "suppressShowKeyBindingsNotice": true
    },
    "gitlens.historyExplorer.enabled": true,
    "javascript.updateImportsOnFileMove.enabled": "always",
    "prettier.singleQuote": true,
    "prettier.trailingComma": "all",
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        {
            "language": "vue",
            "autoFix": true
        }
    ]
}

```

快捷键

```

// 将键绑定放入此文件中以覆盖默认值
[
  {
    "key": "cmd+s",
    "command": "workbench.action.files.saveAll"
  },
  {
    "key": "alt+cmd+s",
    "command": "-workbench.action.files.saveAll"
  },
  {
    "key": "alt+cmd+s",
    "command": "workbench.action.files.save"
  },
  {
    "key": "cmd+s",
    "command": "-workbench.action.files.save"
  },
  {
    "key": "shift+alt+r",
    "command": "eslint.executeAutofix",
    "when": "editorTextFocus && !editorReadonly"
  },
  {
    "key": "alt+cmd+/",
    "command": "editor.action.blockComment",
    "when": "editorTextFocus && !editorReadonly"
  },
]

```