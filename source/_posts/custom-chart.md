## 前言：

项目中遇到一个需求，将Y轴数据源中的不同字段用对应的自定义图标展示，正好Echarts支持自定义系列图表，实现起来挺方便~

所以我又自己做了一个小栗子，总结一下Echarts自定义系列的用法。

这是下文例子的成品demo，包含两个系列(series)的Echarts图，其中一个是普通的折线图，另一个则是根据数据自定义的系列(不同数据用不同图标展示)。

> 源码传送门: [点我🤠](https://github.com/WolfGirlM/EchartsDemos)



![](https://user-gold-cdn.xitu.io/2020/7/9/173314931f08b53e?w=966&h=711&f=png&s=18986)
## 一、Echarts自定义系列简介

在Echarts中，每个series(系列)可以通过type决定是哪种图表类型，官方目前支持[22种不同的type类型](https://echarts.apache.org/zh/option.html#series)。自定义类型`custom`属于其中的一种。

自定义系列支持开发者自定义图表中需要渲染哪种图形元素，核心API如下:
```
series: [
  {
    type: custom, //指定系列类型为自定义类型
    name: '自定义系列', //指定系列的名称
    renderItem: renderItem,//配置自定义系列的内容
    data: data
  }
]
```
其中，最重要的配置项为`renderItem`配置项(类型为函数)，图表的每一项数据在进行渲染时都会调用该函数，并返回自定义的图表内容：
```
  let renderItem = function(params, api) {
      console.log(params, "renderItem_params");
      console.log(api, "renderItem_api");
  }
 ```
 
## 二、自定义系列配置项说明

###  renderItem形参说明

renderItem支持两个参数: `params`、`api`：

`params`参数返回每项data数据项的信息及坐标系的信息:

![](https://user-gold-cdn.xitu.io/2020/7/9/17333c65631da75d?w=868&h=511&f=png&s=40211)

`api`参数主要返回一些方法API([文档传送门](https://echarts.apache.org/zh/option.html#series-custom.renderItem.arguments.api)):

![](https://user-gold-cdn.xitu.io/2020/7/9/17333cd3525e5039?w=657&h=309&f=png&s=26823)

其中，**最常用**的两个API为：

> api.value()--取出 dataItem 中的数值

> api.coord()--进行坐标转换计算

例如series的data项数据为[1,2,3,4]，打印出api.value()的值如下:

![](https://user-gold-cdn.xitu.io/2020/7/9/17333ce8458eeb3c?w=188&h=115&f=png&s=2917)


api.coord()最常用的功能为将api.value()获取到的数值转换为对应的坐标点，以便将图形绘制到图表中：


![](https://user-gold-cdn.xitu.io/2020/7/9/17333ced65d49d16?w=699&h=116&f=png&s=15132)

### renderItem-return项说明

renderItem通过`return`关键字返回一个**图形对象**，并在其中指定具体的图形内容，如果什么都不渲染，可以不返回任何东西(直接return)。
```
 let renderItem = function(params, api) {
     return {
       type: '', //指定图形的类型
       //指定图形的尺寸、坐标轴相对位置
       shape: {
          x: x, 
          y: y,
          width: width,
          height: height
       },
       //指定图形的细节样式
       style: api.style({
          lineWidth: 1
       }) 
     }
  }
 ```
 
## 三、实战


场景假设：series中的data数据为[1,2,3,4],现在为每个数据项设置不同的自定义图标。即数字1到4需要用相应的图标渲染至图表中。

首先，指定两个系列，将其中一个系列类型设置为自定义类型(其中类型为折线图的系列只作为普通图表作为对照，不需要过多关注。）

```
let series = [
    {
      name: "普通折线",
      type: "line",
      data: [1, 2, 3, 4]
    },
    {
      name: "自定义系列",
      type: "custom",
      renderItem: renderItem,
      yAxisIndex: 1,
      data: [1, 2, 3, 4]
    },
 ];

```
 
 接下来，到了重头戏环节，配置`renderItem`函数:
  
 **第一步，利用坐标转换API将数据转换为坐标轴对应的点的位置:**
 ```
  let renderItem = function(params, api) {
      let point = api.coord([api.value(0), api.value(1)]);
    }
 ```
 
 **第二步，利用获取数据的API进行判断，并指定对应的数据所需渲染的图标内容**
 
 本例中，选取`return_path`的图标渲染方式，即使用 `SVG PathData` 做图标的路径进行渲染，这种方式可以用来绘制第三方图标及其他各种图形。
 
 比如，从[阿里巴巴矢量图标库](https://www.iconfont.cn/)中引入SVG图标的方式如下：
 
 步骤1： 从官网复制SVG代码
 
![](https://user-gold-cdn.xitu.io/2020/7/9/17333ed1deaffc9a?w=643&h=560&f=png&s=41591)

 步骤2：将代码中path标签里的`d属性`中的内容提取出来，此即为我们需要的SVG路径
 
 
 
![](https://user-gold-cdn.xitu.io/2020/7/9/17333f0cb4007a11?w=1003&h=140&f=png&s=24804)

指定好图标内容后，可以利用`api.style()`对图标再进行细节的样式优化：
 
 ```
 let renderItem = function(params, api) {
    let point = api.coord([api.value(0), api.value(1)]);
    //根据data中的数据指定需要渲染的图标
    if (api.value(0) == 1) {
      return {
        //图标类型为path类型，通过SVG代码引入第三方图标
        type: "path",
        shape: {
          pathData: "",
          x: params.coordSys.x / 30,
          y: params.coordSys.y / 60,
          width: 30,
          height: 30,
        },
        position: point,
        style: api.style({
          fill: "red",
          lineWidth: 1,
        }),
      };
    }
}
 
 ```
 
重复上述逻辑，即可完成对数据进行自定义图标的渲染啦~~
 
 
 
 
> 源码传送门: [点我🤠](https://github.com/WolfGirlM/EchartsDemos)
