---
title: Markdown 扩展/Markdown-Extention
published: 2026-03-01
description: 'Markdown 扩展语法(Fuwari)'
image: ''
tags: [Markdown, Fuwari]
category: Markdown
draft: false 
lang: zh_CN
pinned: true
---
## GitHub 仓库卡片
你可以添加动态卡片，这些卡片链接到 GitHub 仓库，在页面加载时，仓库信息会从 GitHub API 获取

::github{repo="niuzhix/fuwari"}

使用代码 `::github{repo="<owner>/<repo>"}` 创建一个 GitHub 仓库卡片

```markdown
::github{repo="saicaca/fuwari"}
```

## 警告信息

支持以下类型的警告信息：`note` `tip` `important` `warning` `caution`

:::note
突出用户即使快速浏览也应注意的信息 
:::

:::tip
帮助用户更成功的可选信息 
:::

:::important
用户成功所必需的重要信息 
:::

:::warning
由于潜在风险需要立即引起用户注意的关键内容 
:::

:::caution
某个操作可能产生的负面后果 
:::

### 基本语法

```markdown
:::note
突出用户即使快速浏览也应注意的信息 
:::

:::tip
帮助用户更成功的可选信息 
:::
```

### 自定义标题

警告信息的标题可以自定义

:::note[我的自定义标题]
这是一个带有自定义标题的笔记 
:::

```markdown
:::note[我的自定义标题]
这是一个带有自定义标题的笔记 
:::
```

### GitHub 语法

> [!TIP]
> 也支持 [GitHub 语法](https://github.com/orgs/community/discussions/16925) 

```
> [!NOTE]
> 也支持 GitHub 语法

> [!TIP]
> 也支持 GitHub 语法 
```

### 剧透

您可以在文本中添加剧透内容，文本也支持 **Markdown** 语法

内容 :spoiler[是隐藏的 **ayyy**]!

```markdown
内容 :spoiler[是隐藏的 **ayyy**]!
```