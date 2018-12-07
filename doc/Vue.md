# Vue.js

## Vue实例

###计算属性

```js
new Vue({
    el:"#app",
    computed:{
        mes:function(){
            return ...
        }
    }
})
```

类似于为mes定义了一个getter方法。当方法内的其他data内的参数没有发生变动，mes将会被缓存起来，不会再调用getter方法来重新计算mes的值。

~~部分场景可以替代watch，watch可以用于操作比较耗时或异步请求~~

```js
new Vue({
    el:"#app",
    data:{
        firstname:'',
        lastname:'',
        fullname:''
    }
    watch:{
        firstname:function(newValue){
            this.fullname=newValue+','+this.lastname
        },
        lastname:function(newValue){
        	this.fullname=this.firstname+','+newValue        
        }
    }
})
```

计算属性，默认只有getter。也可以设置setter.

```js
new Vue({
    el:"#app",
    computed:{
        fullname:{
            get:function(){},
            set:function(newValue){}
        }
	}
})
```

###监听属性

watch。可以监听`data`中声明的数据的变化。还可以监听其他对象的变化，比如路由地址的变化。

```js
new Vue({
    el:"#app",
    data:{},
    watch:{
        
    }
})
```

## 指令

- v-for。列表渲染。也可以用于遍历一个对象的所有属性。建议尽可能与key一起使用。
- v-model。在表单`<input>`、`<teatarea>`、`<select>`元素上创建双向数据绑定。
- v-if、v-else、v-else-if。控制内容的显示与否，只有第一次为true时才渲染，为false时会销毁子组件和事件监听器。不推荐同时与v-for使用
- v-show。控制内容的显示与否，只是通过css控制，总会渲染。
- v-bind。用于绑定css。还可以传递props
- v-on。用于绑定事件，可以简写为`@click=xxx()`。还可以绑定按键事件，`v-on:keyup.13='doit()'`。或者`v-on:keyup.enter='doit()'`。

### 自定义指令

```js
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```

也可以局部注册。

### Vue实例的生命周期



## 组件

一个组件通常包含三个部分，HTML、Javascript、css。

```html
<template>
    <div class="red">
        <!--xxxx-->
    </div>
</template>
<script>
    export default{
        data() {
            return {
                
            }
        },
        method:{
            
        }
        }
    }
</script>
<style>
    .red{}
</style>
```

组件使用，必须先注册，也就是说在根实例（`new Vue`）之前必须注册。注册分两种：

- 全局注册。`Vue.component`。

可以使用`Vue.component('name',{data:function(){},template:'html'})`的形式来定义一个组件。

- 局部注册。`var component={name:'',data:{},method:{}}`

`Vue.component`接受与`new Vue`一样的参数，`data`、`computed`、`watch`、`methods`以及声明周期回调。不同的是没有`el`根实例。

组件可以复用。组件内的变量独立，互不影响。

定义组件时，`data`参数必须时函数，即这种形式`data:function(){return {name:'xxx',age:33}}`。

- 字符串
- 单文件组件（.Vue）
- `<script type='text/x-template'></script>`

组件的组织形式：

![](https://cn.vuejs.org/images/components.png)



### 组件传参

`props`。比如：

```js
Vue.component('',{
    props:['title'],
    template:'<h3>{{title}}</h3>'
})
```

单向的数据流，从父组件流向子组件。不应该在子组件修改props中的值。对象和数组是通过引用传递的，子组件修改会影响到父组件中。

如果需要后续修改传入的参数，一种方式是在data中定义一个属性，初始值采用传入的参数。另一种方式是定义一个计算属性。

props定义的时候还可以指定一个或多个类型。也可以指定默认值。还可以设置一个`validator`用于验证传入的参数是否符合要求。

### 自定义事件

通过组件传参，可以从父组件向子组件传递参数达到通信的目的。而通过自定义事件，可以实现子组件向父组件传参。`this.$emit('事件名称',参数…)`。父组件可以在使用子组件的时候，监听自定义事件并绑定到一个方法上。

###插槽

`slot`允许组件中插入一段html。比如：

```html
<my-component>
    <div>
       Hello
    </div>
</my-component>
<script>
    Vue.component('my-component',{
        template:'<div><slot></slot></div>'
    })
</script>
```

如果需要插入多个插槽，可以为插槽定义名称。插槽内也可以使用数据。

### 动态组件

`<component :is='组件名称'></component>`。每次动态组件切换时，都会被销毁重建。在有些场景中并不希望销毁而是缓存起来，这就可以使用`<keep-alive/>`标签将动态组件包裹起来。

### 异步组件

```js
Vue.component('',function(resolve,reject){
    resolve({
        template:''
    })
})
```

可以与webpack的code-splitting配合使用

## 过渡动画

Vue在插入、更新、删除DOM时，提供多种过渡动画。

`transition`组件：

1. 如果有对应的class。在恰当的时机添加/删除class
   1. 进入：
      1. v-enter
      2. v-enter-active
      3. v-enter-to
   2. 离开
      1. v-leave
      2. v-leave-active
      3. v-leave-to
2. 如果有钩子函数，调用钩子函数

## 渲染函数

```js
Vue.component('',{
    render:function(createElement){
        return createElement('h1',this.post)
    }
})
```

Babel有一个插件，使Vue中使用JSX语法。

## 插件

插件能为Vue添加全局功能。在实例化Vue之前，需要调用`Vue.use(Plugin)`。use方法会调用插件的`install`方法。

### Vue-Router路由

实现SPA页面之间的切换。

- Hash模式。浏览器的地址会带/#!。支持所有的浏览器
- History模式。依赖HTML5 History API。
- Abstract模式。支持所有JavaScript运行环境。非浏览器环境会强制使用该模式，比如electron（桌面环境）、cordova（app环境）

组件由路由激活，被初始化时，会被注入route实例。

#### 使用路由

1. 安装依赖， `npm install vue-router`
2. 引用，`import VueRouter from 'vue-router'`
3. Vue设置启用路由，`Vue.use(VueRouter)`
4. 创建路由匹配规则，

```js
const router=new VueRouter({
    routes:[{path:'url',name:'',component:组件}]
})
```

5. 路由匹配规则注册到Vue根实例中。`new Vue({router:router,data:{},methods:{}})`
6. `<router-view/>`作为路由组件的容器，类似于占位符。`<router-link to></router-link>`作为路由链接。

#### 动态路由匹配

动态路径以`:`作为参数的开头。比如`/user/:id`。在组件内部可以通过`this.$route.params`来获取到参数。参数可以设置多个，比如`/user/:username/post/:post_id`。`<router-link :to="{name:'',params:{id:1}}/>"`方式。

多个URL匹配到同一个组件，组件不会再经历销毁重建的过程，生命周期函数不会被调用。但是可以通过`watch`来监听`$route`对象的变化，或者使用`beforeRouteUpdate`导航守卫。

还支持更复杂的匹配模式，或自定义正则匹配模式。

#### 路由传参

除了上面URL路径中的参数，还可以通过查询参数。`$route.query`获取URL中查询参数。`<router-link :to="{name:'',query:{id:11}}" />`

#### 路由切换动画

使用`<transition>`包裹`<router-view>`。还可以为不同的组件设置不同的动画。

#### 嵌套路由

```
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

在子组件中也可以使用`<router-view>`。在路由匹配规则定义的地方，添加`children`节点。

```js
import User from ''
import Post from ''
const router=new VueRouter({
    routes[{path:'/user/:id',component:User,
    children:[
    {
    path:'post ',   //完整的URL为/user/1/post
    component:Post
}
    ]]
})
```

给路由设置name属性，这个路由匹配规则就称之为命名路由。

给`<router-view/>`设置name属性，则称之为命名视图。命名视图允许一个组件内定义多个`<router-view />`。常用于布局。为什么要用路由做布局？

#### 生命周期回调

- 全局回调。使用VueRouter实例定义。
  - beforeEach
  - afterEach
- 路由回调。在定义路由匹配规则时定义。
- 组件回调。可以在组件内定义
  - beforeRouteEnter
  - beforeRouteLeave

### 自定义插件开发

```js
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```

## Vue-Loader

为wepack提供支持处理`.vue`的文件。

## Vuex状态管理

对于很多组件共享数据的场景，vuex应运而生。

对于小型简单的应用，也需要共享数据的场景，除了使用vuex之外，还可以自行实现sotre模式。我们可以定义一些独立的数据对象，各个组件都可以访问这个数据对象，来达到共享数据的目的。数据对象内部维护数据，控制数据访问的方式来防止数据流动的不可控，调试难等问题。

Flux架构？

## Vue-Cli

通过npm安装。能方便生成一个脚手架工程。

`vue create project_name`创建一个项目。`vue create -n project_name`跳过创建git。`vue ui`以图形化的方式来创建项目。

- vue config。审查和查询全局的配置。目录在用户目录下的`.vuerc`。

`vue.config.js`是一个可选的配置文件。