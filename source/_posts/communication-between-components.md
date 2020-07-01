---
title: Vue组件间通信方式
tags: Vue
categories: 前端
---

### 一、组件间存在的关系

通常来说，各组件之间的关系分为:

• 父子组件
• 兄弟组件
• 隔代组件

问题来了，在Vue中，如何界定各组件之间属于什么关系？

在Vue中，组件以树的形式互相联结:

![](https://cdn.nlark.com/yuque/0/2020/png/419980/1585711162304-8f9254cc-cd00-4309-9de4-b9f1b6899a8d.png#align=left&display=inline&height=261&margin=%5Bobject%20Object%5D&name=image.png&originHeight=261&originWidth=690&size=16103&status=done&style=none&width=690)

个人的理解是，组件之间的引入关系决定了各自之间的从属关系:

对于父子组件，如果组件B在组件A中通过以下方式注册,且在组件A的template以如下方式引用：

```javascript
//组件A-父组件
<template>
  <div>
	  <ComponentB></ComponentB>
	  <ComponentC></ComponentC>
  </div>
</template>

<script>
import ComponentB from './ComponentB.vue'
import ComponentC from './ComponentC.vue'
export default {
  components: {
    ComponentB,
    ComponentC
  },
  // ...
}
</script>
```

```javascript
//组件B-子组件
<template>
  <div>
	  <p>How are you</p>
  </div>
</template>

<script>
export default {
  // ...
}
</script>
```

```javascript
//组件C
<template>
  <div>
	  <p>I am fine</p>
  </div>
</template>

<script>
export default {
  // ...
}
</script>
```

则组件A为`父组件`，组件B为`子组件`；同理，A组件与C组件也为`父子组件`。

B组件和C组件都在A组件的template中被引用，所以B、C为`兄弟组件`。其余情况的复杂关系，称为`隔代组件`.

实际开发中，可以将组件之间的关系分为两大类：

第一类： 有引入关系，父子组件
第二类： 没有引入关系，非父子组件

### 二、父子组件通信

#### 2.1 子组件获取/访问父组件中的数据

##### 法1: `Prop`

在父组件中可传递动态或静态的prop给子组件，子组件中通过props接收数据：

例如，父组件A需要传递message字段给子组件B:

```javascript
//组件A-父组件
<template>
  <div>
    // 父组件中采用v-bind形式传递动态Prop
    <ComponentB :message="message"></ComponentB>
  </div>
</template>

<script>
import ComponentB from './ComponentB.vue'
export default {
  components: {
    ComponentB
  },
  data () {
    return {
        message: '123'
    }
  }
}
</script>
```

子组件B接收数据:

```javascript
//组件B-子组件
<template>
  <div>
      <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  props: ['message'],
  data () {
    return {
      ...
    }
  }
}
</script>
```

**由于Prop的单向数据流特性，子组件中不能直接修改props的值。**

如果遇到必须在子组件中改变props的情况，官方文档给出了[2种示例方案](https://cn.vuejs.org/v2/guide/components-props.html)。

##### 法2: `$parent`

`$parent`是Vue提供的API,用它可以在子组件中访问父组件的实例：

```bash
console.log(this.$parent)
```

##### 以上方法的使用场景:

a. `Props`： 大多数情况下，可以优先考虑使用
b. `$parent`： 子组件需要调用父组件的方法或属性，并且并不涉及到修改父组件的数据时可以考虑使用(**Vue是单向数据流动，所以不建议通过使用`$parent`修改父级组件的数据**)

官方也强调了`$parent`的使用局限性:

![](https://img-blog.csdnimg.cn/20200404102713976.png)

此外，层级太深的时候，使用$parent并不方便。

#### 2.2 父组件获取/访问子组件中的数据

##### 法1：`$emit`触发事件

子组件通过调用 `$emit `方法 并传入事件名称及特定的参数(如果需要的话)来触发一个事件：

例如，子组件需要给父组件传回一个布尔值,以便告知父组件中的内容是否显示：

```javascript
//组件B-子组件
<template>
  <div>
    <p @click="open">回到主页面</p>
  </div>
</template>

<script>
export default {
  data () {
    return {
      ...
    }
  }
  methods: {
    open() {
      //changeSomething为需要传递给父组件的事件名称，false为需要传递的参数
      this.$emit('changeSomething', false)
    }
  }  
}
</script>
```

父级组件通过` v-on `监听子组件的事件,并获取子组件传递的参数：

```javascript
//组件A-父组件
<template>
  <div>
    //父组件v-on监听的事件名称与子组件中定义的事件名称一致
      <ComponentB
         v-if="showFlag" 
         @changeSomething="changeStatus">
      </ComponentB>
  </div>
</template>

<script>
import ComponentB from './ComponentB.vue'
export default {
  components: {
    ComponentB
  },
  data () {
    return {
      showFlag: true
    }
  }
  methods: {
    changeStatus(data) {
      this.showFlag = data;
    }
  }  
}
</script>
```

##### 法2： `$ref`

可以通过`$ref `API 为子组件赋予一个 ID 引用，如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例。

##### 法3:  ` $children`
 
`$children`可以获取当前实例的直接子组件上的属性,用法类似与`$parent`

##### 以上方法的使用场景:

a.`$emit`:  大多数情况下，可以优先考虑使用
b.`$refs`、`$children`:  本身都不是响应式的,若只需要单纯获取数据而不需要数据绑定，可考虑用这两种方式

### 三、非父子组件通信

##### 方法1: eventBus

示例场景: 组件B做了相关操作后，组件C中的某个数据需要发生相应变化(B和C不存在引入的关系)

**第一步，在项目中初始化eventBus,以便能在组件中引入:**

```javascript
// main.js
//初始化全局的eventBus
import Vue from 'vue'
Vue.prototype.$EventBus = new Vue()
```

**第二步：在组件B中通过this.$eventBus.$emit触发事件(可传递参数）**

```javascript
//组件B
<template>
  <div>
     <p @click="clickMe">我是B组件</p>
  </div>
</template>

<script>
export default {
  data () {
    return {
        ...
    }
  }
  methods: {
      clickMe() {
         this.$eventBus.$emit('changeC', msg)
    }
  }  
}
</script>
```

**第三步：C组件中通过 this.$eventBus.$on监听来自B组件的事件**

```javascript
//组件c-
<template>
  <div>
      <p>我是C组件</p>
  </div>
</template>

<script>
export default {
  data () {
    return {
        ...
    }
  }
  mounted() {
      this.$eventBus.$on('changeC', (data) => {
        console.log(data)//B组件传过来的msg
    })
  }
  //移除移除自定义事件监听器
  destroyed() {
      this.$eventBus.$off('changeC');
  }
}
</script>
```

**注意，为了避免事件被重复触发，需要组件销毁时，通过 `$off`API移除事件监听。**

##### 方法二：`provide/inject`

此方法类似于Props，适用于存在引入关系，但嵌套层级较深的隔代组件，例如爷孙组件:  A、B为父子组件，B、C为父子组件，当C组件需要获取A组件的数据时，`provide/inject`就体现出了优势：

**第一步，A组件中通过`provide`指定需要传递给子孙组件C的数据：**

```javascript
//组件A-父组件
<template>
  <div>
    <ComponentB></ComponentB>
  </div>
</template>

<script>
import ComponentB from './ComponentB.vue'
export default {
  components: {
    ComponentB
  },
  provide () {
    return {
      message: 'i am here with you'
    }
  }
  data () {
    return {
    }
  }
  methods: {
  }  
}
</script>
```

**第二步：子孙组件C通过inject接收数据**

```javascript
//组件c
<template>
  <div>
    <p>我是C组件</p>
  </div>
</template>

<script>
export default {
  inject: [ "message" ],
  data () {
    return {
      ...
    }
  }
  mounted() {
    console.log(this.message);
  }
}
</script>
```

此种方法的缺点，官方解释如下:

![](https://img-blog.csdnimg.cn/20200404103939396.png)

##### 方法三： `$attrs`/`$listeners`

`$attrs`包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外）。

`$listeners`包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件。

此处实践的并不多，所以不在此处描(luan)述(shuo)了，日后有深入的了解后再进行总结。

##### 方法四:    `Vuex`

关于Vuex的语法在此不做展开赘述，当两个没有直接引入关系的两个组件需要通信时，利用Vuex进行通信如下:

```javascript
//组件B
<template>
 <div>
 <button @click="changeC">ClickMe</button>
 <p>{{ MessageFromC }}</p>
 </div>
</template>

<script>
 export default {
  data() {
    return {
      MessageFromB: 'B组件中的数据'
    }
  },
  computed: {
    MessageFromC() {
      // 从store里获取的C组件的数据
      return this.$store.state.MsgFromC
    }
  },
  methods: {
    changeC() {
      // 通过分发B组件中的改动，将B组件的数据存放至store
      this.$store.commit('receiveBMsg', {
        MsgFromB: this.MessageFromB
      })
    }
  }
}
</script>
```

```javascript
//组件C
<template>
  <div>
    <button @click="changeB">传递数据给B组件</button>
    <p>{{ MessageFromB }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
     	MessageFromC: 'C组件中的数据'
    }
  },
  computed: {
    MessageFromB() {
      // 这里存储从store里获取的B组件的数据
   		return this.$store.state.MessageFromB
    }
  },
  methods: {
    changeB() {
      // 将C组件的数据存放至store，B组件中的数据随之发生变化
      this.$store.commit('receiveMsgFromC', {
      MsgFromC: this.MessageFromC
    })
    }
  }
}
</script>
```

##### 以上方法使用场景:

1. eventBus:  更适用于普通跨级别的组件间通信，但千万注意要移除事件监听。
2. `provide`和 `inject` : 主要在开发高阶插件/组件库时使用。并不推荐用于普通应用程序代码中。(provide 和 inject自身默认不是响应式的)。关于高阶组件，可以参考[这位博主](https://juejin.im/entry/5a524420f265da3e2e6252c5)的博文。
3. `$attrs`/`$listeners`:   适用于高级别的组件(比如封装组件时)
4. Vuex:  适用于模块间耦合性高的项目，一方面方便对数据进行管理，另一方面也能进行跨组件的通信(数据的更改是响应式的)。
