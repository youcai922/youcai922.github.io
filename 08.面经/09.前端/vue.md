#### Vue生命周期

![](https://youcai922.github.io/99.src/img/vue生命周期.webp)

Vue 的生命周期总共分为8个阶段：创建前/后，载入前/后，更新前/后，销毁前/后。

1、beforeCreate（创建前）

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

#### v-if和v-show

- 手段：v-if是动态的向DOM树内添加或者删除DOM元素；v-show是通过设置DOM元素的display:none;样式属性控制显隐：
- 编译过程：v-if切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件；v-show只是简单的基于css切
- 编译条件：v-if是惰性的，如果初始条件为假，则什么也不做；只有在条件第一次变为真时才开始局部编译（编译被缓存？编译被缓存后，然后再切换的时候进行局部卸载）；v-show是在任何条件下（首次条件是为真）都被编详，然后被缓存，而且DOM元素保留；
- 性能消耗：v-if有更高的切换消耗：v-show有更高的初始渲染消耗；
- 使用场景：v-if适合运用条件不大可能改变：v．show适合频繁切换