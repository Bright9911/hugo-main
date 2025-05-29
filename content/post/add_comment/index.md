+++
title = '【Hugo】文章添加评论功能'
date = 2025-05-28T08:10:04+08:00
draft = true
comments= true
categories = [
    "hugo",
    "comment",
    "笔记",
]
image = "1606179652749.jpeg"

+++

<blockquote class="alert-note">

- *评论功能，由 [giscus](https://giscus.app/) 提供支持*
  </blockquote>

## 一、准备工作

1. **注册 GitHub 账号** ：[点击这里](https://github.com/)注册一个 GitHub 账号。
2. **创建博客仓库** ：假设你的 GitHub 用户名是 `Bright9911`，创建一个名为 `Bright9911.github.io` 的仓库用于存放博客代码。
3. **创建评论仓库** ：创建一个名为 `Bright9911/blog-comments` 的仓库用于存储评论数据。

## 二、启用 Discussions 功能

1. 进入评论仓库 `Bright9911/blog-comments`。
2. 点击 “Settings”（设置）。
3. 选择 “General”（通用）。
4. 滚动到 “Features”（功能）部分，勾选 “Discussions”（讨论）。
5. 点击页面底部的 “Save”（保存）。

## 三、安装 Giscus App

1. 打开 [Giscus GitHub App 页面](https://github.com/apps/giscus)。
2. 点击 “Install”（安装）按钮。
3. 在 “Repository access”（仓库访问权限）部分，选择评论仓库 `Bright9911/blog-comments`。

## 四、获取 Giscus 配置代码

1. 打开 [Giscus 配置页面](https://giscus.app/zh-CN)。

2. 填写以下信息：

   仓库名字：`Bright9911/blog-comments`

   Discussion 分类：**Genenal** 或 **announcements**

3. 然后页面会生成一段代码，类似如下：

   ```html
   <script
     src="https://giscus.app/client.js"
     data-repo="Bright9911/blog-comments"
     data-repo-id="MDEwOlJlcG9zaXRvcnkxMjM0NTY3ODg="
     data-category="Discussions"
     data-category-id="MDE4OkNhdGVnb3J5MTE1MDk3MDQ="
     data-mapping="pathname"
     data-reactions-enabled="1"
     data-emit-metadata="0"
     data-theme="light"
     data-lang="zh-CN"
     crossorigin="anonymous"
     async
   ></script>
   ```

   

## 五、修改 Hugo 配置文件

1. 打开博客的 `config.toml` 文件。

2. 新增或修改以下内容：

   ```toml
   [params.comments.giscus]
   repo = "Bright9911/blog-comments"
   repoId = "MDEwOlJlcG9zaXRvcnkxMjM0NTY3ODg="
   category = "Discussions"
   categoryId = "MDE4OkNhdGVnb3J5MTE1MDk3MDQ="
   mapping = "pathname"
   reactionsEnabled = "1"
   emitMetadata = "0"
   theme = "light"
   lang = "zh-CN"
   ```

   

## 六、创建 Giscus 评论组件

1. 在 `layouts/partials` 目录下创建 `giscus.html` 文件。

2. 将生成的代码复制到该文件中：

   ```html
   <script
     src="https://giscus.app/client.js"
     data-repo="{{ .Site.Params.comments.giscus.repo }}"
     data-repo-id="{{ .Site.Params.comments.giscus.repoId }}"
     data-category="{{ .Site.Params.comments.giscus.category }}"
     data-category-id="{{ .Site.Params.comments.giscus.categoryId }}"
     data-mapping="{{ .Site.Params.comments.giscus.mapping }}"
     data-reactions-enabled="{{ .Site.Params.comments.giscus.reactionsEnabled }}"
     data-emit-metadata="{{ .Site.Params.comments.giscus.emitMetadata }}"
     data-theme="{{ .Site.Params.comments.giscus.theme }}"
     data-lang="{{ .Site.Params.comments.giscus.lang }}"
     crossorigin="anonymous"
     async
   ></script>
   ```

   

## 七、在文章中显示评论组件

1. 打开 `layouts/_default/single.html` 文件。

2. 在合适的位置添加以下代码，一般放在文章内容的后面：

   ```html
   {{ if not (eq .Params.comments false) }}
   <div class="comments">
     {{ partial "giscus.html" . }}
   </div>
   {{ end }}
   ```

   

## 八、默认启用评论

1. 打开 `archetypes/default.md` 文件。

2. 添加 `comments: true`，如下所示：

   ```yaml
   ---
   comments: true
   ---
   ```



## 九、管理评论

Giscus 是基于 GitHub Discussions 的评论系统，管理评论主要在对应的 GitHub 仓库中进行。要删除评论，可按以下步骤操作：

<blockquote class="alert-tip">

1. 登录你的 GitHub 账号，进入设置了 Giscus 评论功能的仓库。
2. 找到与你要删除评论相关的 Discussion。Giscus 会根据页面的映射规则（如 URL、pathname 等）将评论关联到对应的 Discussion 中。
3. 在该 Discussion 中找到要删除的评论，点击评论旁边的删除按钮（通常是一个垃圾桶图标）。
4. 确认删除操作，即可完成评论删除。

</blockquote>

此外，你还可以在 GitHub 上进行其他评论管理操作，如对评论进行筛选、标记重要评论等。同时，你可以在仓库的设置中，对 Discussion 的权限和规则进行配置，以更好地管理评论内容



## 十、邮箱提醒

Giscus 评论可以设置提醒，你可以通过以下方式实现：

<blockquote class="alert-warn">

- 设置 GitHub 邮件提醒
  - 登录你的 GitHub 账号，进入设置了 Giscus 评论功能的仓库。
  - 点击仓库右上角的 “设置”（Settings）按钮。
  - 在左侧菜单中选择 “通知”（Notifications）选项。
  - 在 “通知” 设置页面，你可以选择接收通知的方式，如邮件通知，并填写你希望接收通知的邮箱地址。这样，每当有新的评论（包括对评论的回复）时，你都会收到一封邮件通知。

</blockquote>