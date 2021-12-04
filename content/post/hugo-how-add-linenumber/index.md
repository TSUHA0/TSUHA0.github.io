---
title: "Hugo代码片段添加复制按钮，同时避免复制行号"
date: 2021-12-04T14:23:32+08:00
draft: false
tags: ["Hugo"]
categories: ["杂技浅尝"]
toc : true
---





 如何使用hugo和GitHub pages新建博客已经有了很多教程，这里不再赘述，本文说明如何为为hugo渲染的代码片段添加行号，添加复制按钮，同时避免复制按钮复制行号



<!--more-->



## 添加copy按钮，在static目录下添加css和js文件



```shell
➜  static git:(main) tree
.
├── code.svg
├── css
│   └── add-copy-btn.css
└── js
    └── add-copy-btn.js
```

其中 `add-copy-btn.js` 是copy按钮代码的代码逻辑部分

```js
(function() {
  'use strict';

  if(!document.queryCommandSupported('copy')) {
    return;
  }

  function flashCopyMessage(el, msg) {
    el.textContent = msg;
    setTimeout(function() {
      el.textContent = "Copy";
    }, 1000);
  }

  function selectText(node) {
    var selection = window.getSelection();
    var range = document.createRange();
    range.selectNodeContents(node);
    selection.removeAllRanges();
    selection.addRange(range);
    return selection;
  }

  function addCopyButton(containerEl) {
    var copyBtn = document.createElement("button");
    copyBtn.className = "highlight-copy-btn";
    copyBtn.textContent = "Copy";

    var codeEl = containerEl.firstElementChild;
    copyBtn.addEventListener('click', function() {
      try {
        var selection = selectText(codeEl);
        document.execCommand('copy');
        selection.removeAllRanges();

        flashCopyMessage(copyBtn, 'Copied!')
      } catch(e) {
        console && console.log(e);
        flashCopyMessage(copyBtn, 'Failed :\'(')
      }
    });

    containerEl.appendChild(copyBtn);
  }

  // Add copy button to code blocks
  var highlightBlocks = document.getElementsByClassName('highlight');
  Array.prototype.forEach.call(highlightBlocks, addCopyButton);
})();
```



 `add-copy-btn.css` 是用来修改copy按钮的样式

```css
.highlight {
    position: relative;
}

.highlight pre {
    padding-right: 10px;
    background-color:#f8f8f8 !important;
}

.highlight td:first-child {
    user-select: none;
}

.highlight-copy-btn {
    position: absolute;
    top: 7px;
    right: 7px;
    border: 0;
    border-radius: 4px;
    padding: 1px;
    font-size: 0.8em;
    line-height: 1.8;
    color: #fff;
    background-color: #777;
    min-width: 55px;
    text-align: center;
}

.highlight-copy-btn:hover {
    background-color: #666;
}

```

其中重点是 `.highlight td:first-child` ，hugo渲染出的页面中，代码片段被渲染为 `.highlight` 节点，tr下面有两个子节点，都是td节点，第一个td节点负责行号，第二个负责代码部分。通过 `first-child` 筛选第一个td节点，设置为 `user-select: none`  可以避免copy按钮复制行号，否则按钮会把代码和行号一起复制。

上面的修改同时需要修改 `config.toml` 文件。

在 `[params]` 字段，需要添加如下内容，如果原本就有字段，在最后追加即可

```
[params]
  customCSS = ['add-copy-btn.css'] 
  customJS = ['add-copy-btn.js']
  

[markup]
  [markup.highlight]
    codeFences = true
    guessSyntax = true
    hl_Lines = ""
    lineNoStart = 1
    lineNos = true
    lineNumbersInTable = true
    noClasses = true
    style = "github"
    tabWidth = 4

```



