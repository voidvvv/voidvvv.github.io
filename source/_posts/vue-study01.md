---
title: Vue学习-1
date: 2023-12-18 19:45:09
top: 22
categories:
- web
- vue
tags:
- vue
- frontier
---


> 作为一个java后端开发，最近有时候会用到Vue进行前端开发，特地来补课. 此篇博客作为我学习Vue的一个记录，以及知识汇总.不会把知识点罗列到很细，仅供自己之后参考.

<!-- more -->
# 基础

Vue 中文官网https://cn.vuejs.org/

> Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue
> 被设计为可以自底向上逐层应用。Vue
> 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue
> 也完全能够为复杂的单页应用提供驱动。

## 开始使用
入门开始，官网下载[**Vue.js**](https://cn.vuejs.org/js/vue.js)，导入到我们的html页面中.
在html的body中定义一个div，设置id为app（可以自定义）,在script脚本中，new 一个Vue对象，其中el属性根据id取刚才定义的app对象。代码如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    {{name}}
</div>
</body>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            name:"张三"
        }
    })
</script>
</html>
```

## 基本语法
会发现我们定义的div中显示的是我们定义的vue对象中的属性。这就是一个Vue的入门语法，双大括号，官方称做[**mustache**](https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1&tn=baidu&wd=Vue%20mustache&oq=Vue&rsv_pq=fe42335a00002f18&rsv_t=17eaouGpLbNXKYoRVkLeqSKXakpDUEvleZGq%2FLvR%2FwOJ9hBRV4m77vj%2FgaE&rqlang=cn&rsv_enter=1&rsv_dl=tb&rsv_sug3=10&rsv_sug1=7&rsv_sug7=100&rsv_sug2=0&rsv_btype=t&inputT=1618&rsv_sug4=1761)语法。
最开始的定义Vue对象方法，其实跟java中创建对象比较相似。我认为可以将其看作一个构造函数，该构造函数需要传入一个对象，格式是json格式。这样就好理解了。

### vue对象内容：
vue对象中，包含一些属性。
el：指定对应标签id
data：当前vue对象的数据
methods：当前对象的方法
computed：当前对象的计算属性。可以当做属性用，但是是通过方法得出来的属性
component:当前vue对象的组件
created:初始化vue时执行的方法（钩子函数）[^1]。类似于java中的构造方法
[^1]: 这里建议自行百度 vue 生命周期和钩子函数，有更加详细的介绍。这里只是满足我自己使用，故不作详细阐述


### 基本命令
#### v-on: 
 标识给对应组件添加对应的事件。 v-on:[event].[des] 表示给当前组件添加对应事件，并且被对应修饰符[^2]修饰。示例如下：
 [^2]: 修饰符可参考[这里](https://blog.csdn.net/wqliuj/article/details/108654103)
 ```html
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    <button v-on:click="alert">点我</button>
</div>
</body>
<script>
    let app = new Vue({
        el:'#app',
        methods:{
            alert:function (){
                alert("你点击了我！")
            }
        }
    })
</script>
</html>
 ```
 就是给一个按钮添加了点击事件，里面传入方法名，对应方法需要在vue对象中定义好.同样的可以定义别的事件，来触发自定义逻辑
 语法糖：@[event],
 #### v-bind:
 可以给当前标签添加各种属性。例如class，style，key等等。我目前用的也不是很熟练，可以参考[这里](https://blog.csdn.net/qq_39207948/article/details/80938972)，当然，这个
 #### v-for:
 循环遍历某个指定元素，并且生成若干当前标签。共有三种写法。示例如下：
 ```html
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    遍历集合
    <div v-for="book in books">{{book}}</div>
    ---------------------------
    遍历集合，并取出下标
    <div v-for="(book,index) in books">{{book}}--{{index}}</div>
    -----------------------------
    遍历对象，取出对象中的键值对以及下标
    <div v-for="(key,value,index) in job">{{key}}--{{value}}--{{index}}</div>
</div>
</body>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            job:{
                place:"上海",
                work:"上班",
                des:"赚钱"
            },
            books:[
                {
                    name:"十万个为什么",
                    price:10
                },
                {
                    name:"二十万个为什么",
                    price:20
                },
                {
                    name:"三十万个为什么",
                    price:30
                },
            ]
        }
    })
</script>
</html>
 ```
 语法糖：:[data]
#### v-model
绑定vue对象的值到某个标签上。这个指令会自带双向绑定效果。相当于v-on+v-bind结合。示例如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    <input v-model="text">
    {{text}}
</div>
</body>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            text:"阳光明媚"
        }
    })
</script>
</html>
```
这里会发现，当我在输入框中修改text字符串内容时，下面展示的text也会随之改变。这就是双向绑定.
#### v-if，v-else-if，v-else
根据条件来选择展示哪一个标签.示例如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    <div v-if="happy">我开心</div>
    <div v-else="happy">我不开心</div>
    <button @click="change">心情</button>
</div>
</body>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            happy:true
        },
        methods:{
            change:function (){
                this.happy = !this.happy
            }
        }
    })
</script>
</html>
```
### 组件化开发
组件化开发是vue比较重要的一部分。组件化可以把某部分代码实现可重复使用的目的。简化开发流程
组件化开发大致分为三步，1. 构造组件，2注册组件，3使用组件。
其中，全局组件和部分组件的区别在于作用域不一样。可以类比于Java中的**静态属性(方法)**以及**非静态属性(方法)**
其中，组件也有自己的data，methods等等。但是需要注意，组件自己的data是一个方法，返回一个对象，这个对象中可以自己定义属性
#### 全局组件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
    <link rel="stylesheet" type="text/css" href="../css/main.css">
</head>
<body>

<div id="app">
    {{message}}
    <cpn></cpn>
</div>

</body>
<script>
    // 组件创建
    let CPN = Vue.extend({
        template: `<div>
<input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
        </div>`,
        data(){
            return {
                childm:"今天天气不错！"
            }
        },
        methods:{
            alert:function (){
                alert(this.childm)
            }
        }
    })

    //组件注册
    Vue.component('cpn',CPN);

    let app = new Vue({
        el: '#app',
        data: {
            message: "This is Massage"
        }
    })
</script>
</html>
```
构造组件就是Vue.extend()方法，需要传入一个参数，就是一个对象。核心属性为template
注册组件时，方法：Vue.component('cpn',CPN);，需要传入两个参数，一个组件标签名称，一个组件对象.这种方式注册的组件，在当前所有vue对象中均可以使用
使用组件时候，在vue对象管理的标签内部直接使用注册的标签名称即可.
#### 局部组件
直接看代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
    <link rel="stylesheet" type="text/css" href="../css/main.css">
</head>
<body>

<div id="app">
    {{message}}
    <cpn></cpn>
</div>

</body>
<script>
    // 组件创建
    let CPN = Vue.extend({
        template: `<div>
<input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
        </div>`,
        data(){
            return {
                childm:"今天天气不错！"
            }
        },
        methods:{
            alert:function (){
                alert(this.childm)
            }
        }
    })

    //组件注册
    // Vue.component('cpn',CPN);

    let app = new Vue({
        el: '#app',
        data: {
            message: "This is Massage"
        },
        components:{
            cpn:CPN
        }
    })
</script>
</html>
```
可以看到仅仅是注册方法不一样了，局部组件注册方法就是在vue内部声明这个组件。然后使用这个组件。需要注意的是，这种方法注册的组件，只能在当前vue对象中使用，在别的vue对象中是无法识别的
#### 组件语法糖
可以将构造组件的步骤，以及注册组件的步骤合并。直接在注册组件的时候传入构造组件的参数.示例如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
    <link rel="stylesheet" type="text/css" href="../css/main.css">
</head>
<body>

<div id="app">
    {{message}}
    <cpn></cpn>
</div>

</body>
<script>


    //组件注册
    // Vue.component('cpn',CPN);

    let app = new Vue({
        el: '#app',
        data: {
            message: "This is Massage"
        },
        components:{
            cpn:{
                template: `<div>
<input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
        </div>`,
                data(){
                    return {
                        childm:"今天天气不错！"
                    }
                },
                methods:{
                    alert:function (){
                        alert(this.childm)
                    }
                }
            }
        }
    })
</script>
</html>
```
#### 组件模板抽离
将html语句与方法语句写在一起显然不是一个优雅的方法。幸好vue提供了相应的解决方法。就是模板抽离。我可以在html区域定义模板，在vue中使用模板作为组件.
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
    <link rel="stylesheet" type="text/css" href="../css/main.css">
</head>
<body>

<div id="app">
    {{message}}
    <cpn></cpn>
</div>

<template id="test01">
    <div>
        <input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
    </div>
</template>

</body>
<script>


    //组件注册
    // Vue.component('cpn',CPN);

    let app = new Vue({
        el: '#app',
        data: {
            message: "This is Massage"
        },
        components:{
            cpn:{
                template: `#test01`,
                data(){
                    return {
                        childm:"今天天气不错！"
                    }
                },
                methods:{
                    alert:function (){
                        alert(this.childm)
                    }
                }
            }
        }
    })
</script>
</html>
```
上面就是把cpn组件的模板拆到了html中，下面直接使用id获取模板.
#### 父子组件
父子组件，顾名思义，就是组件之间的上下级关系。在vue中这个关系表示为注册区域。
比如A组件在B组件中使用component注册，那么A组件就是B组件的子组件.

##### 父子组件之间的通信 - 父传子
需要在子组件中定义一个props属性来接收父组件传入的数据。可以定义为集合，里面装有接收父组件数据的key值字符串集合。
子组件,在props中定义了一个father属性，来代表其父组件是哪个：
```html
// 组件创建
    let CPN1 = Vue.extend({
        template: `<div>
        <span>我是子组件</span>
        <br>
        子组件获取到的父组件data：{{father}}
        <br>
<input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
        </div>`,
        props:["father"],
        data(){
            return {
                childm:"我是子组件！"
            }
        },
        methods:{
            alert:function (){
                alert(this.childm)
            }
        }
    })
```
父组件，注册子组件。引用子组件时，使用v-bind给子组件添加属性，属性名称即为子组件中props中的属性名称，来将其传递给子组件：
```html
let CPN2 = Vue.extend({
        template: `<div>
        <span>我是父组件</span>
        <br>
        <child1 v-bind:father="name"></child1>
        <br>
<input v-model="childm" placeholder="填写！">
        <button @click="alert">点击我</button>
        </div>`,

        data(){
            return {
                childm:"我是父组件！",
                name:"父组件01"
            }
        },
        methods:{
            alert:function (){
                alert(this.childm)
            }
        },
        components:{
            child1:CPN1
        }
    })
```
总结：父子组件之间的通信-父传子有点类似于java中面向对象的思想，从当前对象中，将属性传递给另外的对象.
其中，这个props还有很多种写法：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210401105302441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Fib3JkZQ==,size_16,color_FFFFFF,t_70)
##### 父子组件之间的通信 - 子传父
子组件传跟父组件通信，是通过事件来通信的。
主要方法如下：
```html
this.$emit('trans-cli',message)
```
子组件中调用$emit，来发射一个事件。其中可以有两个参数，第一个参数就是事件名称，第二个就是事件参数。示例如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="../js/vue.js"></script>
</head>
<body>
<div id="app">
    <input v-model="message">
    <ch1 @trans-cli="alert"></ch1>

</div>

<template id="ch1">
    <div>
       子组件
        <input v-model="cmessage">
        <button @click="trans(cmessage)">给父组件发送消息！！！</button>
    </div>
</template>
</body>
<script>
    var v =  new Vue({
        el:'#app',
        data:{
            message:'我是父组件，你好'
        },
        methods: {
            alert:function (m){
                alert("收到了来自子组件的事件！"+m)
            }
        },
        components:{
            ch1:{
                template:'#ch1',
                data(){
                    return{
                        cmessage:"子组件消息！"
                    }
                },
                methods:{
                    trans:function (message){
                        this.$emit('trans-cli',message)
                    }
                }
            }
        }
    })
</script>
</html>
```
##### 父子组件之间的通信 - 父组件找子组件对象
在开发的时候，有时候需要再父组件中，找到所有子组件对象。有如下几种方法：
1.  在父组件中，调用this.$children,获取当前vue对象的所有子组件集合。获得的是一个数组，可以通过遍历该数组来获取每一个子组件.但是缺点很明显，只能通过下标来获取组件。
2. 在父组件中，通过this.$refs获取所有标有ref属性的子组件。返回一个对象（map），标签中ref的值为key，该对象为值得map。获取指定子组件的话，指定ref值即可。并且经过实测，若有多个相同ref的组件，那么后面的组件会顶替掉前面的
```html
this.$refs.[refName] //其中，refName 就是组件上明明的ref名称
```

##### 父子组件之间的通信 - 子组件找父组件对象
因为一个子组件只可能有一个父组件，所以这个还是比较简单的，直接使用$parent即可获取父组件的vue实例。
##### 父子组件之间的通信 - 寻找根组件
直接使用$root，获取根节点的vue对象。

### 插槽
在模板中，有些时候，我并不需要事先定义好模板中的标签具体的内容，我是想让父组件来决定如何显示这一块，那么就可以用到模板了。直接用slot在模板内定义插槽，在使用时，在标签内放入自定义内容
```html
<template>
<!--    这是一个插槽，可以使用该模板时，定义其内容，也可以内置一个默认样式-->
    <slot>我是默认样式</slot>
</template>

<!-- 使用插槽-->
<tmp1>
    <div >父组件样式</div>
</tmp1>
```
#### 具名插槽
当然，一个模板可以拥有多个插槽。但是如果不设置每个插槽的名字（默认名字default），那么使用起来就会及其不方便。比如，我有两个插槽，A插槽和B插槽。我想在A插槽显示一个按钮，B插槽显示一段话，如下面
```html
<template>
<!--    这是一个插槽，可以使用该模板时，定义其内容，也可以内置一个默认样式-->
    <slot>A插槽</slot>
    <slot>B插槽</slot>
</template>
<!-- 使用插槽-->
<tmp1>
    <div ><button>A插槽放按钮</button></div>
    <div >B插槽放文字</div>
</tmp1>
```
就会出现这种情况：![在这里插入图片描述](https://img-blog.csdnimg.cn/20210402091623531.png)
将两个标签同时放在了A插槽和B插槽.
这时就需要给每个插槽来一个name属性。然后，在父组件中，插入插槽的时候，根据name属性选择插在哪里,如下：
```html
    <tem1>
        <template v-slot:a>
            <div>
                <button>A插槽放按钮</button>
            </div>
        </template>
        <template v-slot:b>
            <div>B插槽放文字</div>
        </template>

    </tem1>
```
需要给每一个单独插入插槽的元素定义template标签，使用v-slot属性，定义插槽名称指定插入位置
#### 插槽中使用子组件属性
在插入插槽时，本质上是在父组件内操作的。所以使用双大括号获取的数据，也都是父组件的。但是此时想要获取子组件数据，就需要在子组件内，通过v-bind来将子组件属性绑定到一个自定义标签属性中，然后在父组件获取.
```html
<template id="tmp1">
    <!--    slot 标签内，用name，外部想要插入插槽就需要使用slot-->
    <!--    slot标签内用 v-slot:name 外部想要插入插槽就需要使用name-->

<!--   slot 标签内，用name，外部想要插入插槽,需要用v-slot:[name] 后面根name名称,来表示对应插槽.然后也可以更改插槽内容，可以用v-slot:[name]=''
来将子组件这个插槽抽出来当做一个对象，用这个对象获取该插槽通过bind属性绑定的值-->
<!--    作用域插槽：给插槽命名，然后bind绑定属性 -->
    <div>
        <!--        具名插槽-->
        <slot name="s"  :pbooks="books">
            <div v-for="book in books">{{book}}</div>
        </slot>
    </div>
</template>
    <tmp1>
        <template v-slot:s="m1">
            <div v-for="(book,index) in m1.pbooks">{{book}}!!!!{{index}}</div>
        </template>
    </tmp1>
```