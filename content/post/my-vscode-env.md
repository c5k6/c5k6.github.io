---
title:       "VSCode 常用快捷鍵"
subtitle:    ""
description: "最常用的Vscode快捷鍵"
date:        2022-03-12
author:      "Jeffery"
tags:        ["VSCode", "快捷鍵"]
categories:  ["實戰系列", "VSCode" ]
---
## VScode 實用快捷鍵
Vscode功能很強大，本篇不做過多描述，如果想要了解Vscode快捷鍵，最好是去 [官網](https://code.visualstudio.com/docs/getstarted/keybindings 本地查詢設定: Ctrl+K Ctrl+S去看本地端有什麼設定。

快捷鍵實在太多，這邊分享個人常用的快捷鍵設定:

### 編輯

比較常用的是這幾個，用於快速排版程式碼。

| command        | key            | command id                        |
|----------------|----------------|-----------------------------------|
| Move Line Down | Alt+Down       | editor.action.moveLinesDownAction |
| Move Line Up   | Alt+Up         | editor.action.moveLinesUpAction   |
| Copy Line Down | Shift+Alt+Down | editor.action.copyLinesDownAction |
| Copy Line Up   | Shift+Alt+Up   | editor.action.copyLinesUpAction   |


### 導航

通常要看代碼的時候，會是這樣的流程:

1. `Ctrl+P`快速開啟檔案
2. `Ctrl+Shift+O`找function  或是 `Ctrl+G` 去到特定的行號(通常是log來的)
3. 然後可能會需要搜尋一些關鍵字，可以用Alt+Left or Right去快速跳轉。

| command                    | key          | command id                       |
|----------------------------|--------------|----------------------------------|
| Go to Symbol...            | Ctrl+Shift+O | workbench.action.gotoSymbol      |
| Go to File...,  Quick Open | Ctrl+P       | workbench.action.quickOpen       |
| Go to Line...              | Ctrl+G       | workbench.action.gotoLine        |
| Go Back                    | Alt+Left     | workbench.action.navigateBack    |
| Go Forward                 | Alt+Right    | workbench.action.navigateForward |

先基本熟悉這些快捷鍵，Vscode就會相當好用了。