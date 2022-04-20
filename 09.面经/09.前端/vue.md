#### Vue生命周期

<img src="https://youcai922.github.io/99.src/img/vue生命周期.webp" alt="vue生命周期" style="zoom:50%;" />

```vue
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>vue生命周期学习</title>
  <script src="https://cdn.bootcss.com/vue/2.4.2/vue.js"></script>
</head>
<body>
  <div id="app">
    <h1>{{message}}</h1>
  </div>
</body>
<script>
  var vm = new Vue({
    el: '#app',
    data: {
      message: 'Vue的生命周期'
    },
    beforeCreate: function() {
      console.group('------beforeCreate创建前状态------');
      console.log("%c%s", "color:red" , "el     : " + this.$el); //undefined
      console.log("%c%s", "color:red","data   : " + this.$data); //undefined 
      console.log("%c%s", "color:red","message: " + this.message) 
    },
    created: function() {
      console.group('------created创建完毕状态------');
      console.log("%c%s", "color:red","el     : " + this.$el); //undefined
      console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化 
      console.log("%c%s", "color:red","message: " + this.message); //已被初始化
    },
    beforeMount: function() {
      console.group('------beforeMount挂载前状态------');
      console.log("%c%s", "color:red","el     : " + (this.$el)); //已被初始化
      console.log(this.$el);
      console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化  
      console.log("%c%s", "color:red","message: " + this.message); //已被初始化  
    },
    mounted: function() {
      console.group('------mounted 挂载结束状态------');
      console.log("%c%s", "color:red","el     : " + this.$el); //已被初始化
      console.log(this.$el);    
      console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化
      console.log("%c%s", "color:red","message: " + this.message); //已被初始化 
    },
    beforeUpdate: function () {
      console.group('beforeUpdate 更新前状态===============》');
      console.log("%c%s", "color:red","el     : " + this.$el);
      console.log(this.$el);   
      console.log("%c%s", "color:red","data   : " + this.$data); 
      console.log("%c%s", "color:red","message: " + this.message); 
    },
    updated: function () {
      console.group('updated 更新完成状态===============》');
      console.log("%c%s", "color:red","el     : " + this.$el);
      console.log(this.$el); 
      console.log("%c%s", "color:red","data   : " + this.$data); 
      console.log("%c%s", "color:red","message: " + this.message); 
    },
    beforeDestroy: function () {
      console.group('beforeDestroy 销毁前状态===============》');
      console.log("%c%s", "color:red","el     : " + this.$el);
      console.log(this.$el);    
      console.log("%c%s", "color:red","data   : " + this.$data); 
      console.log("%c%s", "color:red","message: " + this.message); 
    },
    destroyed: function () {
      console.group('destroyed 销毁完成状态===============》');
      console.log("%c%s", "color:red","el     : " + this.$el);
      console.log(this.$el);  
      console.log("%c%s", "color:red","data   : " + this.$data); 
      console.log("%c%s", "color:red","message: " + this.message)
    }
  })
</script>
</html>
```

Vue 的生命周期总共分为8个阶段：创建前/后，载入前/后，更新前/后，销毁前/后。

1、beforeCreate（创建前）:初始化事件，对数据进行观测

表示实例完全被创建出来之前，vue 实例的挂载元素$el和数据对象 data 都为 undefined，还未初始化。

2、created（创建后）

数据对象 data 已存在，可以调用 methods 中的方法，操作 data 中的数据，但 dom 未生成，$el 未存在 。

3、beforeMount（挂载前）

vue 实例的 $el 和 data 都已初始化，挂载之前为虚拟的 dom节点，模板已经在内存中编辑完成了，但是尚未把模板渲染到页面中。data.message 未替换。

4、mounted（挂载后）

vue 实例挂载完成，data.message 成功渲染。内存中的模板，已经真实的挂载到了页面中，用户已经可以看到渲染好的页面了。实例创建期间的最后一个生命周期函数，当执行完 mounted 就表示，实例已经被完全创建好了，DOM 渲染在 mounted 中就已经完成了。

5、beforeUpdate（更新前）

当 data 变化时，会触发beforeUpdate方法 。data 数据尚未和最新的数据保持同步。

6、updated（更新后）

当 data 变化时，会触发 updated 方法。页面和 data 数据已经保持同步了。

7、beforeDestory（销毁前）

组件销毁之前调用 ，在这一步，实例仍然完全可用。

8、destoryed（销毁后）

组件销毁之后调用，对 data 的改变不会再触发周期函数，vue 实例已解除事件监听和 dom绑定，但 dom 结构依然存在。

#### 父子组件传值

```vue
//父组件
<template>
 <div>
 <h1>我是父组件！</h1>
 <child></child>
 </div>
</template>

<script>
import Child from '../components/child.vue'
export default {
 components: {Child},
}
</script>
```

```vue
//子组件
<template>
 <h3>我是子组件！</h3>
</template>

<script>
</script>
```

父组件通过import的方式导入子组件，并在components属性中注册，然后子组件就可以用标签<child>的形式嵌进父组件了

- prop实现通信

  - 静态传值

  ```vue
  <!-- 父组件 -->
  <template>
   <div>
   <h1>我是父组件！</h1>
   <child message="我是子组件一！"></child> //通过自定义属性传递数据
   </div>
  </template>
  
  <script>
  import Child from '../components/child.vue'
  export default {
   components: {Child},
  }
  </script>
  ```

  ```vue
   <!-- 子组件 -->
  <template>
   <h3>{{message}}</h3>
  </template>
  <script>
   export default {
   props: ['message'] //声明一个自定义的属性
   }
  </script>
  ```

  - 动态传值

  ```vue
   <!-- 父组件 -->
  <template>
   <div>
   <h1>我是父组件！</h1>
   <child message="我是子组件一！"></child>
   <!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
   <child v-bind:message="a+b"></child>
   <!-- 用一个变量进行动态赋值。-->
   <child v-bind:message="msg"></child>
   </div>
  </template>
  
  <script>
  import Child from '../components/child.vue'
  export default {
      components: {Child},
      data() {
          return {
              a:'我是子组件二！',
              b:112233,
              msg: '我是子组件三！'+ Math.random()
          }
      }
  }
  </script>
  ```

  ```vue
   <!-- 子组件 -->
  <template>
  	<h3>{{message}}</h3>
  </template>
  <script>
      export default {
          props: ['message']
      }
  </script>
  ```

- $ref实现通信

ref 是被用来给元素或子组件注册引用信息的。引用信息将会注册在父组件的 $refs 对象上。

1. 如果ref用在子组件上，指向的是组件实例，可以理解为对子组件的索引，通过$ref可能获取到在子组件里定义的属性和方法。
2. 如果ref在普通的 DOM 元素上使用，引用指向的就是 DOM 元素，通过$ref可能获取到该DOM 的属性集合，轻松访问到DOM元素，作用与JQ选择器类似。

```vue
<!-- 父组件 -->
<template>
<div>
    <h1>我是父组件！</h1>
    <child ref="msg"></child>
</div>
</template>

<script>
import Child from '../components/child.vue'
export default {
     components: {Child},
     mounted: function () {
         console.log( this.$refs.msg);
         this.$refs.msg.getMessage('我是子组件一！')
     }
 }
</script>
```

```vue
 <!-- 子组件 -->
<template>
	<h3>{{message}}</h3>
</template>
<script>
export default {
     data(){
         return{
             message:''
         }
     },
     methods:{
         getMessage(m){
             this.message=m;
         }
     }
 }
</script>
```

- $emit 实现通信

  ```
  vm.$emit( event, arg )
  ```

  $emit 绑定一个自定义事件event，当这个这个语句被执行到的时候，就会将参数arg传递给父组件，父组件通过@event监听并接收参数。

```vue
<template>
<div>
    <h1>{{title}}</h1>
    <child @getMessage="showMsg"></child>
</div>
</template>
<script>
 import Child from '../components/child.vue'
 export default {
     components: {Child},
     data(){
         return{
             title:''
         }
     },
     methods:{
         showMsg(title){
             this.title=title;
         }
     }
 }
</script>
```

```vue
<template>
 <h3>我是子组件！</h3>
</template>
<script>
 export default {
     mounted: function () {
         this.$emit('getMessage', '我是父组件！')
     }
 }
</script>
```



#### v-if和v-show

- 手段：v-if是动态的向DOM树内添加或者删除DOM元素；v-show是通过设置DOM元素的display:none;样式属性控制显隐：
- 编译过程：v-if切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件；v-show只是简单的基于css切
- 编译条件：v-if是惰性的，如果初始条件为假，则什么也不做；只有在条件第一次变为真时才开始局部编译（编译被缓存？编译被缓存后，然后再切换的时候进行局部卸载）；v-show是在任何条件下（首次条件是为真）都被编详，然后被缓存，而且DOM元素保留；
- 性能消耗：v-if有更高的切换消耗：v-show有更高的初始渲染消耗；
- 使用场景：v-if适合运用条件不大可能改变：v．show适合频繁切换



