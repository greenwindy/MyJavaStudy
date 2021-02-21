Vue 提供一套基于 HTML 的模板语法, 允许开发者声明式地将真实 DOM 与 Vue 对象的数据绑定在一起。对应的 Html 代码会根据 Vue 的语法重新编译。

# 起步

1. 导入vue.js：

```java
<script src="./vue.js"></script>
```

2. script 中创建 Vue 对象，绑定 div ：

```java
	<div id="root">{{msg}}</div>

    <script>
        new Vue({
            el:"#root",
            data: {
                msg:123,
                bool: true,
                str: 'zs',
                arr: ["zs", "ls", "wu", "zl"],
                obj: {
                    name: "zs",
                    age: 18
                }
            }
        })
    </script>
```

- 不要创建多个Vue对象

- 只能绑定一个div的id
- 语句逗号结尾

## 插值表达式

`{{}}`在 Vue 中称为插值表达式，使用它可以在视图中显示data中的数据。

- 插值表达式支持JS表达式，但不支持语句和流程控制 (选择、循环)

# V-指令

## v-bind

作用：给一个html属性以参数的形式传值

```java
	<div v-bind:title="msg + num"></div>
    <div :title="msg"></div>
    ......
    	data: {
                num: 22,
                msg: "我是一个div"
            }
```

- 简写形式，省略`v-bind`

## v-model

作用：既能以参数的形式传值给表单元素的value属性，也能通过改变value来改变参数的值

```java
	<input v-model="num">
	<input v-model:value="num">
```

- 双向绑定只能使用在表单元素的value上，自动找到标签的value属性
- 简写形式，省略`:value`

##  v-on

作用：监听一个事件，事件发生后从 Vue 对象里的 Methods 找到对应方法触发。类似于html中的`onclick`，`onchange`属性，区别是后者触发后是从 script 里寻找对应的方法定义。

```java
	<button v-on:click="change"> 改变</button>
	<button @click="change"> 改变</button>
	......
            data: {
               bool: false
            },
            methods: {
                change: function () {
                    // 使用Vue对象的数据一定要加this
                  this.bool = !this.bool
                }
            }
```

- 简写：`v-on:`改成`@`
- 属性后只有函数名没有括号，这点与html对应属性不同

##  v-if，v-else-if，v-else

作用：根据表达式的真假改变元素的存在

```java
    <div v-if="bool1">
        v-if
    </div>
    <div v-else-if="bool2">
        v-else-if
    </div>
    <div v-else>
        v-else
    </div>
	...... 
        data: {
            bool1: true,
            bool2: true
        }
```

- 与v-show的区别：v-if为假的标签不存在于DOM树中，而v-show为假的标签被隐藏了，实际还在DOM树上。

## v-show

作用：改变标签的可见性

```java
    <div v-if="bool">
        <img src="https://dss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=3363295869,2467511306&fm=26&gp=0.jpg">
    </div>
    <button @click="changeimg"> 改变</button>
    ......
        methods: {
            changeimg: function () {
                this.bool = ! this.bool
            }
        }
```

## v-for

作用：循环生成标签及其内容

```java
    <div v-for="(item, index)  of arr" :key="index">
        {{item}}--{{index}}
    </div>
    ......
    	data: {
            arr: ["zs", "ls", "wu", "zl"]
        }
```

- in / of 无区别
- 需要给相应标签添加key属性，用来给底层代码标识循环出来的每一个标签

## v-text，v-html

作用，获取和改变标签中的文本内容，类似于DOM中的`innerText`和`innerHtml`

```java
    <div v-text="obj.name">
    </div>
    <div v-text="obj.age">
    </div>

    <div v-html="obj.name">
    </div>
    <div v-html="obj.age">
    </div>
    
    	data: {
            obj:{
                name: "<b>zs</b>",
                age: 18
            }
        }
```
- 两者的区别在于对文本的标签能否识别

## v-pre

作用：阻止预编译，该标签下的内容不会按照Vue编译，会原样输出

## v-once

作用：只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。这可以用于优化更新性能。

## v-cloak

作用：当网络较慢时，网页还在加载 Vue.js ， Vue 来不及渲染，这时页面就会显示出 Vue 源代码。配合CSS使用，可以避免此类现象。但在大型、工程化的项目中（webpack、vue-router）只有一个空的 div 元素，元素中的内容是通过路由挂载来实现的，这时我们就不需要用到 v-cloak 指令咯。

```
    [v-cloak]{
        display: none;
    }
    ......
    <div v-cloak>
    	{{msg}}
    </div>
```

# 模板

模板中的内容会替换挂载的元素，挂载元素的内容都将被忽略。

```
	<div id="root"></div>
	
	<script>
        // 这样html只需要提供一个入口
        // 其所有的内容都可以在一个Vue对象中实现
        new Vue({
            el: "#root",
            data: {
                msg: "1123"
            },
            template:"<div><div @click='clickdiv'>{{msg}}</div>" +
                "<p @click='clickp'>{{msg}}</p></div>",
            methods: {
                clickdiv: function () {
                    this.msg = "div"
                },
                clickp: function () {
                    this.msg = "p"
                }
            }
        })
    </script>
```

# 组件

组件是可复用的 Vue 实例，且带有一个名字。

## 组件的注册

为了能在模板中使用这些组件，必须先注册组件以便 Vue 能够识别。组件的注册方式有两种：全局注册和局部注册。

### 全局注册

```java
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```

- 全局注册的组件在注册后可以在任意Vue对象中使用

- 一个组件的 `data` 选项必须是一个函数，这样每个实例可以维护一份被返回对象的独立的拷贝
- `Vue.component` 的第一个参数时组件的名字，推荐字母全小写且包含一个连字符的形式，这样会帮助你避免和当前以及未来的 HTML 元素相冲突

## 局部注册

全局注册会造成用户下载的 JavaScript 的无谓的增加，往往不够理想。可以通过一个普通的 JavaScript 对象来定义组件，然后在  Vue 对象的`components` 选项中定义你想要使用的组件。

```
var son1 = {
    template:"<div><div @click='f'>son1</div></div>",
    methods: {
        f: function () {
            alert(123)
        }
    }
}
new Vue({
       el: "#root",
       data: {},
       // 注册组件
       components:{
           x:son1,
       },
       template:"<div><x></x></div>",
       methods: {
           f: function () {
               alert(456)
           }
       }
})
```

- 要使得局部注册的组件在其他组件中可用，也需要在该组件中以`components` 选项的方式定义它

## 组件的使用

可以把组件以类似标签的方式使用它：`<组件名></组件名>`

# 生命周期

Vue实例从创建到销毁，有多个阶段：创建—渲染—更新—销毁。可在创建完毕和渲染完毕后请求数据。

```
created:function () {
    // 写一些向后端请求数据的代码
}
```