---
title: Vue学习-2
date: 2023-12-18 19:45:09
categories:
- web
- vue
tags:
- vue
- frontier
---

> 作为一个java后端开发，最近有时候会用到Vue进行前端开发，特地来补课. 此篇博客作为我学习Vue的一个记录，以及知识汇总.不会把知识点罗列到很细，仅供自己之后参考. 此篇是进阶内容
<!-- more -->
[基础篇](../vue-study01)在这里


# 模块化
前端模块化，类似于java中的类的概念。把某些功能抽离出来封装一个类，然后根据功能需要块来选择使用什么模块，什么方法。
目前了解，前端区分模块主要是文件级别区分，一个文件作为一个模块。某个文件用到别的文件中的内容，就需要导入该文件，对应文件需要导出

```javascript
//commonJS模块化的导入和导出
module.exports = {
    //导出属性
    name:'voidvvv',
    //导出方法
    say(mes){
        alert(mes)
    }
}

//导入
const {name,say} = require('demo.js')

// es6默认导出
export default {
    name:'voidvvv - es6',
    say(mes){
        alert(mes+"es6")
    }
}
// es6 指定导出
export var a = 30;

export function shout(){
    alert("shout!!!!!")
}

//ES6 默认导入
import mydefault from './demo'

//ES6 指定导入
import {a} from './demo'
```

## webpack
一个模块化打包工具。可以类比于java的 打jar包，用来帮助我们整理繁琐的前端代码的各个模块。
不同于打jar包的是，打jar包包括了编译文件到class的这一步，但是前端不需要这一步。前端需要的是对各种资源的整理。比如css样式，js代码，甚至图片文件等资源webpack可以帮助我们整理这些资源，在我们使用的时候直接在html页面引入一个最终js即可。
### webpack安装：
1. 首先需要安装[nodejs](http://nodejs.cn/),其中内置了npm包管理工具
2. 在当前项目下执行```npm init```初始化npm管理
3. 命令行执行```npm install webpack -g```进行全局安装。或者在当前项目下，```npm install webpack --save-dev```进行局部安装
4. 在当前项目下，执行```webpack ./src/main.js ./dist/bundle.js```这一步是执行webpack指令。后面的两个参数，一个是打包程序的入口，也是整个应用启动的地方，类似于java 的main方法。后一个参数代表将所有资源整合打包后，所放在什么地方。我这里放在当前项目的dist文件夹下的bundle.js中了
5. 执行完后，会生成目标文件，在html中引用该文件即可
### webpack.config.js
在每次执行打包命令时，都要指定一遍源文件以及目标文件未免有些过于繁琐，因为这两个位置是不经常发生改变的，所以需要有个地方来配置
在当前文件夹下创建webpack.config.js文件，里面内容如下：
```javascript
const path = require('path')

module.exports = {
    entry:'./src/main.js',
    mode: 'development',
    output:{
        path:path.resolve(__dirname,'dist'),
        filename:'bundle.js'
    }
    }
```
这里就配置了当前项目打包的源文件以及目标文件。
之后，若还想再次打包，直接执行webpack即可.

手动指定配置文件方式：```webpack --config 文件位置```
### loader
现在，我们只是完成了javascript文件的打包。若我们想使用css，甚至图片呢？这就需要webpack的组件：loader了。
若js中引用了css，则需要安装cssloader以及styleloader。详情可看[官网](https://webpack.docschina.org/loaders/)。这里就只介绍怎么用，以及我自己遇到的一些问题以及排查
#### 导入css相关：
首先执行```npm install url-loader --save-dev```以及```npm install --save-dev css-loader```，然后配置webpack.config.js
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```
#### 导入图片相关
图片需要两个loader。一个url ```npm install url-loader --save-dev```，一个file ```npm install file-loader --save-dev```。然后配置webpack.config.js
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: '[path][name][hash:8].[ext]',
            },
          },
        ],
      },
    ],
  },
};
```
这里需要注意一点，就是导入图片只虽然是引入了两个loader，但是在配置文件中只需要一个loader就可以了。配置了两个会出现加载图片重复并且无法显示的问题。
### webpack插件
目前我们的项目打包后，只会生成js代码以及用到的图片等文件.想要自动生成首页的html的话，还需要安装插件html-webpack-plugin
```npm install html-webpack-plugin --save-dev```
在配置文件中导入```const htmlWebpackPlugin = require('html-webpack-plugin')```
然后新增

```javascript
plugins:[
        new htmlWebpackPlugin({
            template:'index.html'
        })
    ],
```

   然后项目在打包的时候，会带上根目录的indexhtml作为模板。并且将打包后生成的目标js自动放进模板中

### webpack-dev-server搭建本地服务器

```npm install webpack-dev-server```安装组件。

配置文件中，添加如下配置：

```json
devServer: {
        contentBase: path.join(__dirname, 'dist'),
        compress: true,
        port: 9000,
        inline:true
    }
```

然后执行 webpack serve 命令启动。

或者可以直接在配置文件中的scripts中配置命令。使用npm run 命令方式启动

老版本的这个，需要的命令是webpack-dev-server。此命令用在新版本会出现错误。

```Error: Cannot find module 'webpack-cli/bin/config-yargs'```

是新老版本不兼容的错误。使用新命令webpack serve即可



### webpack整合vue
首先通过npm安装vue ```npm install vue --save```，这，我们就可以在js文件中，通过import Vue from 'vue'来引入vue了，然后就可以向入门时候一样来使用了。
但是这里会遇到一个问题：
```You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build.```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210403131545445.png)
这是因为默认使用了runtimeonly版本，无法对template编译。所以我们需要进行一部转换(后期template分离后，可以用这个版本，更快，更小),在webpackconfig文件中，添加如下配置：
```script
resolve:{
        alias:{
            'vue$':'vue/dist/vue.esm.js'
        }
    }
```



# 脚手架

帮助进行vue开发的框架。把webpack以及一些插件loader的配置给默认配置好了

首先安装vuecli：```npm install @vue/cli -g```

然后通过vue创建项目。（可以通过webstorm以及相关工具直接创建）```vue create 项目名称```

创建好项目后，就可以直接进行开发了。目前使用的是vue4.+，默认有asset文件，放静态资源，components文件夹，放vue模板，App.vue，默认的一个主模板，main.js，默认的主程序入口。总之，之前所有的配置在这里全部都默化了。可以直接使用。

## 更改默认配置

我们可以看到配置全部都默认化了，而且自带css，图片等的解析功能。并且自带本地服务功能。若是想再修改配置文件，可以通过```vue ui```命令来图形化的修改相关属性，以及管理各种组件。也可以通过新建vue.config.js文件，看当作自己的配置文件。

具体用法跟webpack中的webpack.config.js一样，可以当作sprringboot里面的application.yml

## 路由

vue开发使用的是单页面模式，即整个前端项目只有一个html，通过路由来控制显示什么样的组件。

1. 如果想使用路由，需要先安装```npm install vue-router --save```

2. 然后新建一个router文件夹，创建index.js文件，导入：```import VueRouter from "vue-router";```示例如下

```javascript
// 路由
import VueRouter from "vue-router";
import Vue from 'vue'
import Home from "@/components/Home";
import PicCon from "@/components/PicCon";

//1. Vue.use(插件) 个人理解，算是注册插件
Vue.use(VueRouter)

const routes = [
    {
        path:'/home',
        component:Home
    },
    {
        path:'/pic',
        component:PicCon
    }
]

//2. 创建Vuerouter对象
const router = new VueRouter({
    routes
})

export default router;

```

需要注意的是，<font color='red'>这里的数组routes对象，不可以随便命名。必须为routes</font>

3. 再在main.js中，引入路由，放在我们根vue对象里

```javascript
import Vue from 'vue'
import App from './App.vue'
import './plugins/element.js'
import VueRouter from "@/router";

Vue.config.productionTip = false

new Vue({
  router:VueRouter,
  render: h => h(App),
}).$mount('#app')

```

4. 使用：```<router-link to="home">home</router-link>```这样来表示路由到某个path，

   ```<router-view></router-view>```来展示对应路由显示的内容

5. 若不想使用router-link，想使用自定义的组件来实现，可以使用@click绑定方法.
```vue
this.router.push('url') 来实现。push中也可以放对象，对象中可以有path，以及query参数。
{
    path::'/home',
    query:{
        name:'test',
        age:20
    }
}
```

### 路由懒加载

在需要用到js组件时，才加载相应的组件。类似于java设计模式中的懒汉式。将路由的写法写成让如下方式即可：

```javascript
const routes = [
    {
        path: '/',
        redirect: '/home'
    },
    {
        path: '/home',
        component: () => import('@/components/Home')
    },
    {
        path: '/pic',
        component: () => import   ('@/components/PicCon')
    }
]
```

### 路由嵌套

当一个路由下面有子路由时，需要使用路由嵌套。比如：/home/news 

嵌套路由需要在路由中使用children属性，里面配置子路由

```javascript
{
        path: '/home',
        component: () => import('@/components/Home'),
        children:[
            {
                path:'news',
                component:()=>import('@/components/home/HomeNwes')
            },
            {
                path:'video',
                component:()=>import('@/components/home/HomeVideo')
            }
        ]
    }
```

然后，子路由中，也使用如下方式，相对路由来定位

```vue
    <router-link to="news">嗯，去看看新闻</router-link>
    <router-link to="video">嗯，去看看视频</router-link>
    <router-view>routerview</router-view>
```
问题：如何使用routerlink能实现新打开一个页面来跳转呢？ 直接在router-link 上添加target属性，设置_blank即可
### 路由参数传递
传递方式两种。params和query。
**params** 可以类比于javaweb中的pathvariable获取参数形式。这种方式的操作方法如下：
```javascript
//路由文件配置映射： 
/home/:id
//传递方式：直接在路径拼接

// 接受方式：
在子组件中使用：
this.$route.params.id
// 可以类比于javaweb使用 @PathVariable
```
**query** 可以类比于get请求直接加参数。这种方式会在路径上显示参数（get请求）。这种方式的操作方法如下：
```vue
//路由配置文件无需改动

//传递方式：router-link 标签，其中的to属性需要交给vue管理。即改为 :to. 其中内容也由字符串变为对象。如下：
<router-link :to="{path:'/news',query:{name:'zhangsan',age:10}}">嗯，去看看新闻</router-link>

// 取出参数,在子组件中，使用如下：
this.$route.query 可以取出传入的对象。
this.$route.query.name 可以取出对象中的某个值
```
## 路由全局导航守卫


## keepalive

# vueX vue状态管理工具。
vuex主要就是负责将各个模块公共的一些资源统一进行管理，并且进行响应式修改的工具.
使用：

1. 安装 ```npm install vuex --save```

2. 配置 ：新建store文件夹，创建index.js，内容如下：

   ```javascript
   import Vue from "vue"
   import Vuex from "vuex"
   
   Vue.use(Vuex)
   
   const store = new Vuex.Store({
       state:{
           
       },
       mutations:{
           //此处放方法。外界修改值就是调用这里的方法.方法会默认有一个参数，state，就是本对象中的state,可以通过这个state来修改真正的属性
           //mutations的使用比较特别,需要用 $store.commite('方法名'，参数) 的方法来调用.传入的参数会放在第二个参数
           incres(state){
               
           }
       },
       actions:{},
       getters:{
           //这里用来放公共的计算属性。这里计算属性有两个默认的参数，第一个就是state属性，第二个时getter属性。用来递归
       },
       modules:{}
   })
   
   export default store;
   ```

3. 注册：在main.js中首先导入：```import store from './store'```,然后再放进根vue中：

   ```javascript
   new Vue({
     router:VueRouter,
     store,
     render: h => h(App)
   }).$mount('#app')
   ```

4. 使用： 在子组件中，通过this.$store ,来获取注册的store对象。想获取其中的state，```$store.state.name```即可

# axios



## 整合elementui

```shell
vue add element
```
### elementui使用问题及解决
#### el-form表单校验相关：
基本的写法：
```javascript
<el-form :model="ruleForm" status-icon :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
  <el-form-item label="密码" prop="pass">
    <el-input type="password" v-model="ruleForm.pass" autocomplete="off"></el-input>
  </el-form-item>
  <el-form-item label="确认密码" prop="checkPass">
    <el-input type="password" v-model="ruleForm.checkPass" autocomplete="off"></el-input>
  </el-form-item>
  <el-form-item label="年龄" prop="age">
    <el-input v-model.number="ruleForm.age"></el-input>
  </el-form-item>
  <el-form-item>
    <el-button type="primary" @click="submitForm('ruleForm')">提交</el-button>
    <el-button @click="resetForm('ruleForm')">重置</el-button>
  </el-form-item>
</el-form>
<script>
  export default {
    data() {
      var checkAge = (rule, value, callback) => {
        if (!value) {
          return callback(new Error('年龄不能为空'));
        }
        setTimeout(() => {
          if (!Number.isInteger(value)) {
            callback(new Error('请输入数字值'));
          } else {
            if (value < 18) {
              callback(new Error('必须年满18岁'));
            } else {
              callback();
            }
          }
        }, 1000);
      };
      var validatePass = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请输入密码'));
        } else {
          if (this.ruleForm.checkPass !== '') {
            this.$refs.ruleForm.validateField('checkPass');
          }
          callback();
        }
      };
      var validatePass2 = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请再次输入密码'));
        } else if (value !== this.ruleForm.pass) {
          callback(new Error('两次输入密码不一致!'));
        } else {
          callback();
        }
      };
      return {
        ruleForm: {
          pass: '',
          checkPass: '',
          age: ''
        },
        rules: {
          pass: [
            { validator: validatePass, trigger: 'blur' }
          ],
          checkPass: [
            { validator: validatePass2, trigger: 'blur' }
          ],
          age: [
            { validator: checkAge, trigger: 'blur' }
          ]
        }
      };
    },
    methods: {
      submitForm(formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            alert('submit!');
          } else {
            console.log('error submit!!');
            return false;
          }
        });
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
      }
    }
  }
</script>
```
其中，el-form中的v-model所绑定的对象，需要是vue中的对象。el-form-item中，prop属性所对应的值，就是 el-form中的v-model所绑定的对象 的对应属性名称。
这样，我们定义校验器的时候，就可以使用el-form中的rules属性来指定校验器对象，校验器对象是一个数组。数组中每个对象的key就是el-form中的v-model所绑定的对象 的对应属性名称，表示每个属性的校验方法。
同时，el-form-item也可以使用自己的rules标签来覆盖el-form给的校验规则。
#### el-form动态添加表单且自定义校验规则：
但是，在动态添加表单的时候，如果想自定义校验器该怎么做呢？
```html
<el-form  :model="curEdit"  :rules="rules" ref="ruleForm" label-width="120px">

                    <el-form-item style="width: 70%;" v-for="(item,i) in curEdit.times" :label="'打款时间点'+(i+1)" :prop="'times.'+i+'.value'" :rules="checkRule(i)">
                        <el-input  v-model="item.value"></el-input>
                    </el-form-item>


            </el-form>
```
这是我自定义的表单，可以看到，其中的el-form-item条数是不断变化的。根据的是vue对象中的一个数组来决定的。
那么，这么个动态变化的表单如何做校验呢
首先，这个数组必须在一个vue对象A中，然后该数组成员也必须是有属性的对象。
整个表单el-form来跟这个对象A绑定，v-model，这里可以指定规则，也可以不指定，因为没用。
其次，el-form-item使用v-for来遍历这个对象中的该数组属性。
这里的prop属性，需要为属性值。但是因为我们这里需要取数组中的属性，则先拿到当前数组对应的属性，我这里叫times。然后取索引，i，最后拼接数组成员的属性，我这里直接就叫value。
最重要的一点，rules属性，因为我这里需要自定义实现，并且需要校验的属性是 A 对象的一个数组属性中的一个成员的属性，所以默认的映射也很难达到，这里可以直接 ：prop，这里可以传入一个vue 的方法。方法返回一个校验规则对象即可。
详细的校验规则对象可以取elemntui官网看
我这里方法名就叫checkRule，并且传入了一个索引
下面是我的方法的具体实现。主要就是判断数字是否合规并且是否重复,且是失去焦点才会校验
```javascript
checkRule: function (index) {
            let vali = {
                required: true,
                validator: (rule, value, callback) => {
                    // console.log(value)
                    if (index === 0) {
                        callback();
                        return;
                    }
                    if (!value.match(/^\d{1,2}$/)) {
                        callback(new Error('请输入正确格式的内容后提交'));
                        return;
                    }
                    if (value < 0 || value > 23) {
                        callback(new Error('请输入正确格式的内容后提交'));
                        return;
                    }
                    var count = 0;
                    for (var x = 0; x < this.curEdit.times.length; x++) {
                        if (this.curEdit.times[x].value === value) {
                            count++;
                            if (count >= 2) {
                                callback(new Error('请输入正确格式的内容后提交'));
                                return;
                            }
                        }
                    }
                    callback();
                },
                trigger: 'blur'
            }
            return vali;
        }
```