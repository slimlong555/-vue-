  # 理解Vue的数据响应式
  ## 什么是响应式
  就比如当一个物体对外界的变化做出反应就叫响应式的，如“我打你一拳，你会喊疼”。
  ## Vue的数据响应式
就是对数据做出改变时，视图上也会做出相应的变化。

举个例子

```
1const vm = new Vue({
2    el:'#app',
3    data:{
4        name: 'Luna'
5    }
6})
```
根据以上代码，页面上对应位置会显示Luna，如执行vm.name='Pola'，则会由Luna变成Pola。

**但这是正常情况，还有些变态情况**
* 当data为空对象时，Vue会给出警告，例如。
```
new Vue({
data: {},
template: `
<div>{{n}}</div>
`}).$mount("#app");
```
**会显示n是一个没有被定义的实例**
* 当data为a，却要在页面中显示b，因为Vue只检查第一层，可以消除警告，但不会将它转换为响应式的
```
1new Vue({
 2  data: {
 3    obj: {
 4      a: 0 // obj.a 会被 Vue 监听 & 代理
 5    }
 6  },
 7  template: `
 8    <div>
 9      {{obj.b}}
10      <button @click="setB">set b</button>
11    </div>
12  `,
13  methods: {
14    setB() {
15      this.obj.b = 1; 
16    }
17  }
18}).$mount("#app");
```
  *此时点击 set b，视图中不会显示1，因为Vue没法监听一开始不存在的obj.b*


## 解决方法
1. 提前把key声明好，哪怕是空值。
```
 1new Vue({
 2  data: {
 3    obj: {
 4      a: 0 // obj.a 会被 Vue 监听 & 代理
 5      b: undefined
 6    }
 7  },
 8  template: `
 9    <div>
10      {{obj.b}}
11      <button @click="setB">set b</button>
12    </div>
13  `,
14  methods: {
15    setB() {
16      this.obj.b = 1; 
17    }
18  }
19}).$mount("#app");
```
这是再点击set b就会显示1


2. 使用Vue.set或this.$set
```
 1new Vue({
 2  data: {
 3    obj: {
 4      a: 0 // obj.a 会被 Vue 监听 & 代理
 5      b: undefined
 6    }
 7  },
 8  template: `
 9    <div>
10      {{obj.b}}
11      <button @click="setB">set b</button>
12    </div>
13  `,
14  methods: {
15    setB() {
16      Vue.set(this.obj,'b',1); //也可以这么写this.$set(this.obj,'b',1)
17    }
18  }
19}).$mount("#app");
```
**当data中有数组怎么办，如果数组长度一直增加，就没有办法提前把key都声明出来，每次改数组，都要用Vue.set或this.$set也很麻烦。**

可以直接使用变异方法中的push
```
 1new Vue({
 2  data: {
 3    array: ["a", "b", "c"]
 4  },
 5  template: `
 6    <div>
 7      {{array}}
 8      <button @click="setD">set d</button>
 9    </div>
10  `,
11  methods: {
12    setD() {
13      this.array.push("d");
14    }
15  }
16}).$mount("#app");
```
***注意***：this.$set 作用于数组时，并不会自动添加监听和代理，原因未知，只能问尤雨溪了。
使用 Vue 提供的数组变异 API 时，会自动添加监听和代理。