+++
title = '【Hugo】修改博客背景并引入动态背景'
date = 2025-05-27T15:07:38+08:00
draft = true
categories = [
    "hugo",
    "笔记",
]
image = "6b951400bb98fd5fcd57d2c9599df286.jpg"

+++

## 1、修改背景图

- （1）准备一张背景图，尽可能大一点，并放到`assets/background`文件夹下（不存在则自己创建）

- （2）在页脚文件`layouts/partials/footer/custom.html`中（不存在则自己创建），引入以下代码，修改对应的背景图片名

```html
<style>
  body {
    background: url({{ (resources.Get "background/背景图片名").Permalink }}) no-repeat center top;
    background-size: cover;
    background-attachment: fixed;
  }
</style>
```

## 2、引入动态背景

### 2.1、樱花飞舞

- （1）下载【[sakura.js](https://letere-gzj.github.io/hugo-stack/p/hugo/custom-background/sakura.js)】（Ctrl + S 保存），并放到`assets/background`文件夹下

- （2）在`layouts/partials/footer/custom.html`中，引入以下代码

```html
<script src={{ (resources.Get "background/sakura.js").Permalink }}></script>
```

### 2.2、点线漂浮（particles.js）

- 【[particles.js文档](https://github.com/VincentGarreau/particles.js)】
- （1）前往【[配置页面](http://vincentgarreau.com/particles.js/)】配置参数，参数按自己喜好即可，唯一注意要修改的参数是 **detect_on**，要改成 **window**

- （2）下载配置文件，以及 **particles.js** 所需要的js文件

  [[particlesjs-config.json](https://letere-gzj.github.io/hugo-stack/p/hugo/custom-background/particlesjs-config.json)]（Ctrl + S 保存），本博客的动态背景json配置，有需求的可直接下载

- （3） 把下载好的文件，解压并将以下两个文件放到`assets/background`文件夹下
  - **particlesjs-config.json**
  - **particles.min.js**

- （4）在`layouts/partials/footer/custom.html`中，引入以下代码

  ```html
  <div id="particles-js"></div>
  
  <script src={{ (resources.Get "background/particles.min.js").Permalink }}></script>
  <script>
    particlesJS.load('particles-js', {{ (resources.Get "background/particlesjs-config.json").Permalink }}, function() {
      console.log('particles.js loaded - callback');
    });
  </script>
  
  <style>
    #particles-js {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      z-index: -1;
    }
  </style>
  ```

  