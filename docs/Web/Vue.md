# Vue

学习笔记： https://github.com/fly321/VueLearnNotes（更多用法看笔记）

## ES6

### ES6的对象属性增强型写法

 ES6以前定义一个对象

```js
const name = "zzz";
const age = 18;
const user = {
  name:name,
  age:age
}
console.log(user);
```

 ES6写法

```js
const name = "zzz";
const age = 18;
const user = {
	name,age
}
console.log(user);
```

### ES6对象的函数增强型写法

 ES6之前对象内定义函数

```js
const obj = {
  run:function(){
     console.log("奔跑");
  }
}
```

ES6写法

```js
const obj = {
  run(){
     console.log("奔跑");
  }
}
```

### 高阶函数

**filter过滤函数**

```js
const nums = [2,3,5,1,77,55,100,200]
//要求获取nums中大于50的数
//回调函数会遍历nums中每一个数，传入回调函数，在回调函数中写判断逻辑，返回true则会被数组接收，false会被拒绝
let newNums = nums.filter(function (num) {
  if(num > 50){
    return true;
  }
  return false;
 })
 //可以使用箭头函数简写
//  let newNums = nums.filter(num => num >50)
```

**map高阶函数**

```js
// 要求将已经过滤的新数组每项乘以2
//map函数同样会遍历数组每一项，传入回调函数为参数，num是map遍历的每一项，回调函数function返回值会被添加到新数组中
let newNums2 = newNums.map(function (num) {
  return num * 2
 })
 //简写
//  let newNums2 = newNums.map(num => num * 2)
console.log(newNums2);
```

**reduce高阶函数**

```js
// 3.reduce高阶函数
//要求将newNums2的数组所有数累加
//reduce函数同样会遍历数组每一项，传入回调函数和‘0’为参数，0表示回调函数中preValue初始值为0，回调函数中参数preValue是每一次回调函数function返回的值，currentValue是当前值
//例如数组为[154, 110, 200, 400],则回调函数第一次返回值为0+154=154，第二次preValue为154，返回值为154+110=264，以此类推直到遍历完成
let newNum = newNums2.reduce(function (preValue,currentValue) {
  return preValue + currentValue
 },0)
//简写
// let newNum = newNums2.reduce((preValue,currentValue) => preValue + currentValue)
console.log(newNum);
```

## helloVue

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <title>HelloVuejs</title>
</head>
<body>
  <div id="app">
      <h2>{{message}}</h2>
      <p>{{name}}</p>
  </div>
  <script>
    //let变量/const常量
    //编程范式：声明式编程
    const app = new Vue({
      el:"#app",//用于挂载要管理的元素
      data:{//定义数据
        message:"HelloVuejs",
        name:"zzz"
      }
    })
  </script>
</body>
</html>
```

## 插值操作

### 1、Mustache语法

 在vue对象挂载的dom元素中，`{{}}`不仅可以直接写变量，还可以写简单表达式。可以直接引用data数据

```html
<!-- Mustache的语法不仅可以直接写变量，还可以写简单表达式 -->
    <h2>{{firstName + lastName}}</h2>
    <h2>{{firstName + " " + lastName}}</h2>
    <h2>{{firstName}}{{lastName}}</h2>
    <h2>{{count * 2}}</h2>
```

### 2、v-once

-once表示该dom元素只渲染一次，之后数据改变，不会再次渲染。

```html
  <div id="app">
    <h2>{{message}}</h2>
    <!-- 只会渲染一次，数据改变不会再次渲染 -->
    <h2 v-once>{{message}}</h2>

  </div>
```

### 3、v-html

在某些时候我们不希望直接输出`<a href='http://www.baidu.com'>百度一下</a>`这样的字符串，而输出被html自己转化的超链接。此时可以使用v-html。

```html
<h2 v-html="url"></h2><!-- data中的数据可以是html，用该标签可以将它转换为html格式，而非纯文本 -->
```

### 4、v-text

v-text会覆盖dom元素中的数据，相当于js的innerHTML方法。

```html
<h2 v-text="message">啧啧啧</h2><!--message的值，覆盖 啧啧啧-->
```

### 5、v-cloak

```html
<div id="app" v-cloak>
  <h2>{{message}}</h2>
</div>
```

## 基础语法

1、v-bind	语法糖：v-bind:title = :title

2、v-if系列

- v-if
- v-else-if
- v-else

3、v-for

```vue
<div id="vue"> 
    <li v-for="item in items"> {{ item.message }} </li>
</div>
```

4、v-on 监听事件	语法糖：v-on:click="事件"=@click="事件"

5、**v-model**	双向绑定

```vue
<div id="app">
    <!-- v-bind:value只能进行单向的数据渲染 -->
    <input type="text" v-bind:value="searchMap.keyWord">
    <!-- v-model 可以进行双向的数据绑定 -->
    <input type="text" v-model="searchMap.keyWord">
    <p>您要查询的是：{{searchMap.keyWord}}</p>
</div>
```

input中值改变，data中数据也随之改变

## 计算属性和侦听器

1. 使用Mastache语法拼接`<h2>{{firstName+ " " + lastName}}</h2>`
2. 使用方法methods`<h2>{{getFullName()}}</h2>`
3. 使用计算属性computed`<h2>{{fullName}}</h2>`

methods和computed的区别，**computed有缓存**调用三次只有一次记录，而methods调用三次有三次记录

```js
//计算属性的值会随着data值改变而及时改变
const app = new Vue({
	el:"#app",
    data:{
    	firstName:"skt t1",
    	lastName:"faker"
    },
    computed: {
    	fullName:function(){
          return this.firstName + " " + this.lastName
        }
    },
    methods: {
    	getFullName(){
        return this.firstName + " " + this.lastName
    	}
    },
})
```

> 例子中计算属性computed看起来和方法似乎一样，只是方法调用需要使用()，而计算属性不用，方法取名字一般是动词见名知义，而计算属性是属性是名词，但这只是基本使用。

### setter和getter

```js
  computed: {
    fullName:{
      //计算属性一般没有set方法，只读属性
      set:function(newValue){
        console.log("-----")
        const names = newValue.split(" ")
        this.firstName = names[0]
        this.lastName = names[1]
      },
      get:function(){
        return this.firstName + " " + this.lastName
      }
    }
  }
```

赋值：fullName='wangjian'

但是计算属性一般没有set方法，只读属性，只有get方法，但是上述中newValue就是新的值，也可以使用set方法设置值，但是一般不用。

## 组件

```vue
  <div id="app">
    <!-- 3.使用组件 -->
    <my-cpn></my-cpn>
    <my-cpn></my-cpn>
    <my-cpn></my-cpn>
    <cpnc></cpnc>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    // 1.创建组件构造器对象
    const cpnc = Vue.extend({
      template:`
        <div>
          <h2>标题</h2>
          <p>内容1...<p>
          <p>内容2...<p>
        </div>`
    })
    // 2.注册组件
    Vue.component('my-cpn', cpnc)
    const app = new Vue({
      el:"#app",
      data:{
      },
      components:{//局部组件创建
        cpnc:cpnc
      }
    })
  </script>
```

`template`中是组件的DOM元素内容。

1. 全局注册，通过 `Vue.component `。
2. 局部注册，通过 `components:{cpnc:cpnc}`。

> 组件的data为什么必须要是函数

组件的思想是复用，定义组件当然是把通用的公共的东西抽出来复用

data()函数是局部的，只存储当前组件所需要的数据

data是vue全局的

### 父传子（ props）

 v-bind是 不支持使用驼峰标识的，例如`cUser`要改成`c-User`。

```vue
  <div id="app">
    <!-- v-bind不支持驼峰 :cUser改成 :c-User-->
    <!-- <cpn :cUser="user"></cpn> -->
    <cpn :c-User="user"></cpn>
    <cpn :cuser="user" ></cpn>
  </div>
  <template id="cpn">
    <div>
      <!-- 使用驼峰 -->
      <h2>{{cUser}}</h2>
      <!-- 不使用 -->
      <h2>{{cuser}}</h2>
    </div>
  </template>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
  <script>
    // 父传子：props
    const cpn = {
      template: "#cpn",
      props: { //对象写法
        //驼峰
        cUser:Object,
        //未使用驼峰
        cuser:Object
      },
      data() {return {}},
      methods: {},
    };
    const app = new Vue({
      el: "#app",
      data: {
        user:{
          name:'zzz',
          age:18,
          height:175
        }
      },
      components: {
        cpn
      }
    })
  </script>
```

### 子传父`$emit`

子组件向父组件传值，使用自定义事件`$emit`。

> 父子双向绑定实例

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>组件通信-父子通信案例</title>
</head>

<body>
  <!-- 父组件 -->
  <div id="app">

    <cpn :number1='num1' :number2='num2' @num1change="num1Change" @num2change="num2Change"></cpn>

    <h2>父组件{{num1}}</h2>
    <input type="text" v-model="num1" >
    <h2>父组件{{num2}}</h2>
    <input type="text" v-model="num2">

  </div>

  <!-- 子组件 -->
  <template id="cpn">

    <div>
      <h2>{{number1}}</h2>
      <h2>{{dnumber1}}</h2>
      <input type="text" :value="dnumber1" @input="num1input">
      <h2>{{number2}}</h2>
      <input type="text" :value="dnumber2" @input="num2input">
    </div>
  </template>

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>

  <script>
    // 父传子：props
    const cpn = {
      template: "#cpn",
      data() {
        return {
          dnumber1:this.number1,
          dnumber2:this.number2
        }
      },
      props:{
        number1:[Number,String],
        number2:[Number,String],
      },
      methods: {
        num1input(event){
          this.dnumber1 = event.target.value
          this.$emit('num1change',this.dnumber1)
        },
        num2input(event){
          this.dnumber2 = event.target.value
          this.$emit('num2change',this.dnumber2)
        }
      },
    };
    const app = new Vue({
      el: "#app",
      data() {
        return {
          num1:1,
          num2:2,
        }
      },
      methods: {
        num1Change(value){
          this.num1=value
        },
        num2Change(value){
          this.num1=value
        }
      },
      components: {
        cpn
      },
    })
  </script>
</body>
</html>
```

### 插槽

```vue
<!-- 子组件 -->
<template id="cpn">
 <div>
    <div>
      {{message}}
    </div>
     <!-- 插槽默认值 -->
	<slot><button>button</button></slot>
 </div>
</template>
```

父组件中的标签直接替换slot所在的位置

```vue
<!-- 父组件 -->
<cpn>
	<span style="color:red;">这是插槽内容222</span>
</cpn>
```

> slot-具名插槽的使用

在slot标签中使用**name**属性

```vue
  <!-- 插槽的基本使用使用<slot></slot> -->
  <!-- 子组件模板 -->
  <template id="cpn">
    <div>
      <slot name="left">左边</slot>
      <slot name="center">中间</slot>
      <slot name="right">右边</slot>
      <slot>没有具名的插槽</slot>
    </div>
  </template>
```

在cpn标签内，添加标签，在标签中使用slot的属性指定插槽名称

```vue
  <!-- 父组件 -->
  <div id="app">
    <cpn>
      <span>没具名</span>
      <span slot="left">这是左边具名插槽</span>
      <!-- 新语法 -->
      <template v-slot:center>这是中间具名插槽</template>
      <!-- 新语法缩写 -->
      <template #right>这是右边具名插槽</template>
    </cpn>
  </div
```

