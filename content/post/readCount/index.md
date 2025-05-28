+++
title = '【Hugo】文章浏览量统计'
date = 2025-05-28T07:46:55+08:00
draft = true
categories = [
    "hugo",
    "笔记",
]
image = "2077.jpg"

+++

**前言：**

- 博客文章统计组件个人找到了两个，分别是[**不蒜子**](https://busuanzi.ibruce.info/)，以及[**vercount**](https://cn.vercount.one/)
- **不蒜子**比较老，但很稳定，到现在仍然可以使用，没有停服
- **vercount**则比较新，并做了一些代码优化
- 两种使用都基本一样，差别不大，看自己喜好，下面的教程是以**vercount**来举例，用不蒜子的话，就把对应的脚本和元素标签替换一下就好

## 1、基本引入

- （1）修改`layouts/partials/footer/custom.html`（不存在则自行创建），引入脚本

```html
<!-- vercount的脚本；若用不蒜子，则更换成不蒜子的脚本就好 -->
<script defer src="https://cn.vercount.one/js"></script>
```

- （2）准备一张[**浏览量**](https://letere-gzj.github.io/hugo-stack/p/hugo/view-count/eye.svg)的icon(Ctrl+S保存)，放到`assets/icons`文件夹下

- （3）修改`layouts/article/components/details.html`，在合适的位置下加入以下代码

```html
<div class="article-details">
    ...
    <footer class="article-time">
        ...
        <!-- 浏览量统计 -->
        <div id="viewCount">
            {{ partial "helper/icon" "eye" }}
            <time class="article-time--reading">
                <!-- vercount统计当前页面浏览数的标签；若用不蒜子，更换成不蒜子对应的标签就好 -->
                <span id="vercount_value_page_pv">loading... </span>次
            </time>
        </div>
    </footer>
    ...
</div>
```

- （4）这样文章详情的开头就显示浏览数了

## 2、问题修复

- 问题描述：
  - 博客首页的文章列表显示了浏览数，且只有第一篇文章才有浏览数，并且浏览数的数字不正确



- 产生原因：
  - `layouts/partials/article/components/details.html`此html文件也被用在了博客首页的文章列表，所以也触发了vercount读取当前页面的浏览数
  - 因为读取的数据是当前页面的浏览数，也就是首页的浏览数，并非文章的浏览数，所以数据只加载一次，且不准确



- 解决思路：
  - 由于vercount并未提供`只查询文章浏览数`的接口，只有一个`文章浏览数+1，并且返回浏览数`的接口，所以无法实现首页对每篇文章的浏览数的单独查询
  - 既然无法实现首页展示每篇文章的单独浏览数，那就直接隐藏就好了，等点入文章才看到具体的浏览数



- 具体操作：
  - （1）修改`layouts/partials/footer/custom.html`，引入以下代码

```html
<script>
    function showHideView() {
        // 判断是否存在vercount标签
        let viewCounts = document.querySelectorAll("#viewCount");
        if (viewCounts) {
            // 判断是否为文章页
            let article =  document.querySelector(".article-page");
            if (!article) {
                viewCounts.forEach(ele => {
                    ele.style.display = 'none';
                });
            }
        }
    }
    
    showHideView();
</script>
```

