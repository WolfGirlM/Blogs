---
title: 【SVG.js实战篇】01-Vue中优雅的使用SVG.js
tags: SVG
categories: 前端
---

## 一、关于SVG.js

SVG.js是一个基于SVG的开源JS库，支持操作 SVG 和执行 SVG 动画。真的很好用！(~~在实际项目场景中，使用SVG.js很便利友好~~)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzI2LzE3MzhhNmQyOTJiNjY0OTk?x-oss-process=image/format,png)

* [github仓库](https://github.com/svgdotjs/svg.js)
* [官方文档](https://svgjs.com/docs/3.1/)
*  本文[Demo源码](https://github.com/WolfGirlM/SVGjs-Demos)

## 二、安装

```
// npm
npm install @svgdotjs/svg.js
// yarn
yarn add @svgdotjs/svg.js

```
## 三、项目中引入

```
import { SVG } from "@svgdotjs/svg.js"
```

## 四、绘制基础图形

[](demo地址)

绘制图形之前，首先需要创建一个SVG.js可以用来识别的容器:

```
<template>
  <div>
    <div id="simpleSquare"></div>
  </div>
</template>
```

接下来，可以愉快的开始绘制图形了:

### 第一步: 使用`SVG()`API初始化一个SVG节点实例:

```
let shapeModel = SVG().addTo("#simpleSquare").size("100%", "100%");`

```
### 第二步，利用SVG提供的图形绘制方法绘制基础图形:

**绘制正方形**

```
shapeModel.rect(100, 100).attr({ fill: "#00B1B6" });
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkMjkyYjE5YmNjYzM?x-oss-process=image/format,png)

**绘制圆**
```
 shapeModel.circle(100).radius(50).attr({ fill: "#0284A3" });
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkM2NiMjAwNzU4ZTQ?x-oss-process=image/format,png)

**绘制多边形**

```
 shapeModel.polygon("0,0,100,50,50,100").fill("#175369") .stroke({ width: 1 });
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkNDU0ZTQ3OGRhM2I?x-oss-process=image/format,png)

**绘制线条**
```
 shapeModel.line(0, 0, 100, 150).stroke({ width: 5, color: "#6488B7" });
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkNDU5ZWRhYWI3YTU?x-oss-process=image/format,png)
**绘制自定义图形**
```
 shapeModel.image('图片存储路径');
```
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkNDVkM2I3NGJhNTE?x-oss-process=image/format,png)
**绘制矢量Path路径(SVG的path路径绘制而成的图)**
```
 shapeModel.path("M0 0 H50 A20 20 0 1 0 100 50 v25 C50 125 0 85 0 85 z").fill("#495975");
```

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC83LzE0LzE3MzRkNDYxNjBkNjYxZTA?x-oss-process=image/format,png)

> 本文Demo[源码传送门](https://github.com/WolfGirlM/SVGjs-Demos)

未完待续<(*￣▽￣*)/

