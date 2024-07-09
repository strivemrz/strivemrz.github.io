---
title: vue组件化开发
categories: vue
tags:
  - vue
excerpt: ''
cover: https://pic.quanjing.com/xu/fh/QJ9126056126.jpg?x-oss-process=style/794ws
---

#### Vue基础知识

#### computed(不是函数)会有缓存，只要依赖值不变，不会重新计算。

~~~js
//原始写法
computed:{
    practice:{
        get:function () {
            return this.msg.split('').reverse().join('')
        },
            //一般不用 practice="aaa"
            set:function (data1) {
                console.log(data1)
            }
    }
}
~~~

#### 监听器(当数据改变时触发)

~~~js
watch:{
      msg:function (newValue,oldValue) {
        console.log("新值："+newValue)
        console.log("旧值："+oldValue)
      },
      "user.username":{
        handler:function(newValue,oldValue){
          console.log("watch案例--"+newValue+" "+oldValue)
        },
        deep:true
      },
      "user.password":{
        immediate:true,//初始化时候就加载
        handler:function (newValue,oldValue) {
          console.log("password监听："+newValue+" "+oldValue)
      }
}
~~~

#### 绑定类名

~~~html
<p v-bind:class="{active:isActive,hello:isActive}">test案例</p><!--第一个active类名，第二个isActive是data里面自己设置的boolean数值，当为true时，class生效。-->
~~~

#### v-show和v-if的区别

v-show：只是简单的切换元素的display 带有v-show的元素始终会被渲染并保留在DOM种，频繁切换状态

v-if：只有后面为false，对应的元素以及子元素都不会都不会被渲染，控制DOM元素的创建和销毁，一般用在运行时条件很少改变的情况，一次性。

#### 数组更新检测

~~~html
<template>
  <div id="div1">
    <ul>
      <li v-for="item in persons" :key="item">
        {{item}}<input type="checkbox"/>
      </li>
    </ul>
      <!-- 往persons头部新增一个数值，所选的复选框会向上移一个(默认通过下标移动),当用:key设定主键时，会通过主键匹配 -->
    <button @click="addPerson()">新增值</button>
  </div>
</template>
~~~

#### splice处理数组

~~~html
this.persons.splice(1,0,7,7,7)
<!-- 新增数组内容 1：新增位置，0：删除元素的个数，7...新增数据 -->

this.persons.splice(1,3)
<!-- 删除数组内容 1：删除位置，3：删除元素个数 -->

this.persons.splice(1,3,8,8,8)
<!-- 替换数组内容 1:替换位置，3：替换元素个数， 8...替换内容-->
~~~

#### 事件修饰符

**鼠标事件**

- 阻止事件冒泡(子节点点击事件，不会自动触发父节点点击事件)

~~~html
<template>
  <div id="div1" @click="divCli">
    <button @click.stop="changeSh">改变数组</button>
  </div>
</template>
~~~

- 阻止默认行为(不会触发submit跳转，执行divCli方法)

~~~html
<form action="">
    <input type="submit" value="提交" @click.prevent="divCli">
</form>
~~~

- 点击触发一次

~~~html
<button type="button" @click.once="divCli">点击触发一次</button>
~~~

**键盘事件**

- 键盘按下enter键触发

~~~html
<input type="text" @keyup.enter="divCli">
~~~

#### v-model与checkbox使用

~~~html
<!-- 如果msg为""/null 则只能为true/false -->
<!-- 如果msg为[] 则checkbox选中则添加数据 -->
<div>
    <input type="checkbox"  v-model="msg" value="苹果">苹果
    <input type="checkbox"  v-model="msg" value="鸭梨">鸭梨
    <input type="checkbox"  v-model="msg" value="香蕉">香蕉
    {{msg}}
</div>
~~~

~~~html
<!-- sex:"男" -->
<div>
    <input type="radio" value="男" v-model="sex">
    <input type="radio" value="女" v-model="sex">
    {{sex}}
</div>
~~~

~~~html
<!-- selectNum:["张家口"] -->
<div>
    <select v-model="selectNum" multiple>
        <option value="张家口">张家口</option>
        <option value="清河">清河</option>
    </select>
    {{selectNum}}
</div>
~~~

#### v-model修饰符

~~~html
<!-- 当input输入框失去焦点时，进行msg数据绑定 -->
<div>
    <input type="text" v-model.lazy="msg">
    {{msg}}
</div>
~~~

~~~html
<!-- 把输入的内容转换成number数据 -->
<div>
    <input type="text" v-model.number="msg">
    {{typeof msg}}
</div>
~~~

~~~html
<!-- 将输入内容的两端空格去掉 -->
<div>
    <input type="text" v-model.trim="msg">
    {{msg}}
</div>
~~~

#### 使用vue-cli进行组件化开发

- 在components文件夹中创建Content组件

~~~html
<template>
<div>{{msg}}</div>
</template>
<script>
    export default {
        name: "Content",
        data:function () {
            return{
                msg:"我的第一个组件"
            }
        }
    }
</script>
<style scoped>
</style>
~~~

- 在其他页面调用该组件

~~~html
<script>
    import Content from "./components/Content"
    export default {
        components: {
            content:Content//或者直接Content，在template中使用Content
        }
    }
</script>
<template>
  <div>
    <content/>
  </div>
</template>
~~~

#### 父组件向子组件传值

> 单项数据流，父组件改变子组件也改变，子组件改变，父组件不变

- 父组件传值---调用组件的

~~~html
<template>
    <div>
    <context :message="sex" mess="定值"/> <!-- 还可以传定值 -->
    <context/>
    </div>
</template>
~~~

- 子组件接收并使用值

~~~js
<!-- 使用props接收父值 -->
props:['message']
或
<!-- 根据类型传值 -->
props:{
	message:{
		type:Number,
		default:111,
		require:true
	}
}
~~~

~~~html
<h1>{{message}}</h1>
~~~

#### 子组件向父组件传值

- 子组件定义自定义事件

~~~js
methods:{
	subMess:function () {
<!-- subMes:自定义事件名,msg.name:传的数据 -->
		this.$emit("subMes",this.msg.name)
	}
},
~~~

- 父组件绑定自定义事件

~~~html
<div>
    <!-- @subMes:自定义事件名  addMess:方法名 -->
    <context :message='user.username' @subMes="addMess"/>
</div>
~~~

~~~js
addMess:function (e) {
<!-- 此时e的数据就是子组件msg.name的数据 -->
console.log(e)
}
~~~

#### 父组件访问子组件

- 在子组件标签上标注ref ID

~~~html
<context ref="cont" :message='user.username' @subMes="addMess"/>
~~~

- 在父组件方法中使用子组件内容

~~~js
console.log(this.$refs.cont.msg.name)
~~~

#### slot插槽

> 组件中的内容根据不同的需求而设定

- 在子组件中定义内容

~~~html
<template>
  <div id="wrap_div">
    <h4>我是content组件</h4>
    <slot></slot>
    <button>{{message}}</button>
  </div>
</template>
~~~

- 父组件调用子组件，并传入定义内容

```html
<!-- 使用双标签 -->
<context><input placeholder="输入内容"/></context>
```

#### 具名插槽

> v-slot只能添加在template上
>
> 父级模板里的内容都是在父级作用域中编译的，子模版里的内容都是在子级作用域中编译的。

- 子标签添加slot插槽

~~~html
<template>
    <div class="wrap_div">
        <h4>content组件</h4>
        <slot name="input"></slot>
        <slot name="p"></slot>
        <button>{{message}}</button>
    </div>
</template>
~~~

- 父组件使用具名插槽

~~~html
<!-- v-slot只能用在template标签上 -->
<Content>
    <template  v-slot:input>
		<input placeholder="input输入框">
    </template>
    <template v-slot:p>
		<p>p标签插槽</p>
    </template>
</Content>
~~~

**插槽备用内容**

~~~html
<template>
    <div class="wrap_div">
        <h4>content组件</h4>
        <slot name="input"><input type="text" placeholder="插槽备用内容"></slot><!-- 当没有传值时，用的是备用内容 -->
        <slot name="p"></slot>
        <button>{{message}}</button>
    </div>
</template>
~~~

**作用域插槽的使用**

- 子组件插槽绑定属性值

~~~html
<div class="wrap_div">
        <slot name="input" :list="list"></slot>
</div>
~~~

- 父组件对接子组件插槽

~~~html
<Content>
    <template v-slot:input="lists">
		<ul>
    		<li v-for="item in lists.list">{{item}}</li>
        </ul>
    </template>
</Content>
~~~

#### provide、inject跨域访问祖先数据

**访问静态数据**

**非响应式---修改父组件，子组件不会修改内容**

- 父组件定义provide数据

~~~js
<script>
export default {
  data:function () {
    return{
        message:"hello world"
    }
  },
  components:{
    Content
  },
  provide:{mess:"跨域访问祖先数据"}
}
</script>
~~~

- 子组件接收数据

~~~html
<template>
{{mess}} <!-- 显示跨域访问祖先数据 -->
</template>
<script>
    export default {
        name: "ContentChild",
        methods:{
            alerRoot:function () {
                alert(this.$root.message)
            }
        },
        inject:['mess']
    }
</script>
~~~

**访问父组件动态数据**(父组件provide使用函数形式)

**非响应式---修改父组件，子组件不会修改内容**

- 父组件定义provide数据

~~~js
<script>
export default {
  data:function () {
    return{
        message:"hello world"
    }
  }
  provide:function () {
      return {
          mess:this.message
      }
  }
}
</script>
~~~

- 子组件接收数据

~~~html
<template>
{{mess}} <!-- 显示跨域访问祖先数据 -->
</template>
<script>
    export default {
        name: "ContentChild",
        inject:['mess']
    }
</script>
~~~

**响应式---修改父组件，子组件会修改内容--类似于浅拷贝**

1)父组件返回的对象形式

- 父组件返回的对象形式

~~~js
<script>
export default {
  data:function () {
    return{
        message:{
            mess:"hello world"
        }
    }
  },
  methods:{
        changeUper:function () {
            this.message.mess="修改父组件内容"
        }
  }
  provide:function () {
      return {
          mess:this.message
      }
  }
}
</script>
~~~

- 子组件接收

~~~html
<template>
	{{mess.mess}}
</template>
<script>
export default {
        name: "ContentChild",
        inject:['mess']
    }
</script>
~~~

2)父组件返回函数形式

- 父组件返回函数形式

~~~js
<script>
export default {
  data:function () {
    return{
        message:"hello world"
    }
  },
  provide:function () {
      return {
          mess:()=>this.message
      }
  }
}
</script>
~~~

- 子组件以函数形式接收

~~~html
<template>
    <div>
        孙子组件内容---{{mess()}}
    </div>
</template>

<script>
    export default {
        name: "ContentChild",
        inject:['mess']
    }
</script>
~~~



#### 生命周期函数

~~~text
beforeCreate：在实例生成之前会自动执行的函数
created：在实例生成之后会自动执行的函数
beforeMount：在模板渲染完成之前执行的函数
mounted：在模板渲染完成之后执行的函数

beforeUpdate：页面元素改变之前---显示在页面的元素
updated：页面元素改变之后

beforeUnmount：元素销毁之前---v-if
unmounted：元素销毁之后
~~~

#### 组合式API

> 在组合式API之前，项目的碎片化使的理解和维护复杂组件变得困难。选项的分离掩盖了潜在的逻辑问题，在处理单个逻辑点时，必须不断地跳转相关代码的选项块。
>
> 组合式API使用地方，setup----不能使用this关键字访问数据(在created钩子函数之前执行，此时data数据还没有生成)

**基本使用---非相应式**

~~~html
<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
    <div>
        {{msgs}}
        <button @click="changeMs">change</button>
    </div>
</template>
<script>
    export default {
        setup:function(){
            let ms="setup组合式API"
            console.log("生命周期取代..."+ms)
            function changeMs() {
                ms="改变ms值"
            }
            return{
                msgs:ms,
                changeMs
            }
        }
    }
</script>
----------------------------------------------------------------
响应式---ref对象形式
<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
    <div>
        {{msgs}}
        <button @click="changeMs">change</button>
    </div>
</template>
<script>
    import {ref} from 'vue'
    export default {
        setup:function(){
            let ms=ref("setup组合式API")
            console.log("生命周期取代..."+ms)
            function changeMs() {
                ms.value="改变ms值"
            }
            return{
                msgs:ms,
                changeMs
            }
        }
    }
</script>
响应式---reactive 引用形式
<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
    <div>
        {{obj.token}}
        <button @click="changeObj()">change</button>
    </div>
</template>
<script>
    import {ref,reactive} from 'vue'
    export default {
        setup:function(){
            let obj=reactive({
                token:"hello token"
            })
            function changeObj() {
                obj.token="改变token值"
            }
            return{
                changeObj,
                obj
            }
        }
    }
</script>
~~~

#### 路由

> 改变url路径，但是页面不进行整体刷新
>
> 路由表，是一个映射表，一个路由就是一组映射关系，key:value
>
> vue-router是基于路由和组件的，路由是用来设定访问路径，将路径和组件映射起来

**安装路由**

~~~bash
npm install vue-router
~~~

**配置路由**

- 创建组件

- 在src目录下创建router文件夹，创建index.js路由文件

~~~js
import {createRouter, createWebHashHistory, createWebHistory} from "vue-router"

// 1. 定义路由组件
import child from "../components/ContentChild.vue"
import content from "../components/Content.vue"

// 2. 定义路由配置
const routes = [
    {path:"/child",component: child},
    {path:"/content",component:content}
];

// 3. 创建路由实例
const router = createRouter({
    // 4. 采用hash 模式
    history: createWebHashHistory(),
    // 采用 history 模式
    // history: createWebHistory(),
    routes, // short for `routes: routes`
});
export default router
~~~

- 在main.js中配置路由

~~~js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router/index'

//注意use要在mount之前
createApp(App).use(router).mount('#app')
~~~

- 在App.vue中实现跳转，显示功能

~~~html
<template>
    <div>
        <!-- 跳转，这里写的是path路径 -->
        <router-link to="/child">使用child组件</router-link>
        <router-link to="/content">使用content组件</router-link>

        <!-- 组件内容显示位置 -->
        <router-view></router-view>
    </div>
</template>
~~~

#### 路由传参

- 给路径传参

~~~html
<router-link to="/content/456">使用content组件</router-link>
~~~

- 在路由表路径上设置参数

~~~js
// 2. 定义路由配置
const routes = [
    {path:"/child/:id",component: child},
    {path:"/content/:id",component:content}
];
~~~

- 组件接收参数

~~~js
let id=this.$route.params.id;  //非组合式API

import {useRoute} from 'vue-router' //组合式API
console.log(useRoute().params.id)
~~~

#### 页面找不到，404界面

- 定义404组件
- 路由表配置404组件映射

~~~js
// 2. 定义路由配置
const routes = [
    {path:"/child/:id",component: child},
    {path:"/content/:id",component:content},
    {path:"/:path(.*)",component:notFound} // 这里使用正则表达式匹配所有路径，当前面没有匹配到，就使用404组件
];
~~~

#### 动态路由条件限制(使用正则表达式)

~~~js
//当前路由参数一定是数字
{path:"/child/:id(\\d+)",component: child},
//有多个参数
{path:"/child/:id+",component: child},
//参数可有可无，可重复叠加
{path:"/child/:id*",component: child},
//参数可有可无，参数不可以重复叠加
{path:"/child/:id?",component: child},
~~~

#### 嵌套路由

> 在调用路由组件之后，该组件中还有一层路由，

- 定义路由组件
- 定义路由配置

~~~js
// 2. 定义路由配置
const routes = [
    {
        path:"/child",
        component: child,//路由组件
        children:[
            {
                path:"route1",//这里路径不用加"/"
                component:route1 //子路由组件
            },
            {
                path:"route2",
                component:route2
            }
        ]
    },
    {path:"/:path(.*)",component:notFound}
];
~~~

- 路由组件中还嵌套一个组件

~~~html
<div>
    <p>child组件</p>
    <router-link to="/child/route1">嵌套路由1</router-link>
    <router-link to="/child/route2">嵌套路由2</router-link>
    <router-view></router-view>
</div>
~~~

#### 通过js传参

~~~js
this.$router.push("/")
//通过传入对象
this.$router.push({path:"/"})
//传入参数
this.$router.push({path:"/showRoute1/234"})
this.$router.push({name:"abc",params:{id:123}})//获取：this.$route.params.id
//get方式,带问号
this.$router.push({path:"/showRoute1",query:{id:123}})//获取：this.$route.query.id

//替换当前位置
this.$router.push({path:"/showRoute1",query:{id:123},replace:true})
//前进，后退
this.$router.go(1)
this.$router.go(-1)
~~~

#### 命名路由

**一个路由对应多个组件**

- 路由配置

~~~js
{path:"/showTwo",components:{
default:defRoute"
routeF:route1,
routeS:route2
}},
~~~

- 在渲染位置增加name

~~~html
<template>
    <div>
        <router-link to="/showTwo">使用child组件</router-link>
        <router-view name="routeF"></router-view>
        <router-view name="routeS"></router-view>
        <router-view></router-view>
    </div>
</template>
~~~

#### 路由重定向

~~~js
// 2. 定义路由配置
const routes = [
    {
        path:"/child",
        name:"routeName",
        component: child,
        children:[
            {
                path:"route1", //这里不用加 "/"
                component:route1
            },
            {
                path:"route2",
                component:route2
            }
        ]
    },
    {
        path:"/",
        redirect:"/child"  //路由重定向path
        redirect:{name:"routeName"} //命名路由
    	redirect:(to)=>{ //方法
            return:{path:"/child"}
        }
    }]
~~~

#### 路由组件传参

~~~js
// 2. 定义路由配置
const routes = [
    {
        path:"/child/:id(\\d+)",
        component: child,//路由组件,
        props:true
    },
    {path:"/:path(.*)",component:notFound}
];
~~~

~~~js
<!--child组件-->
<script>
export default {
        props:["id"]
    }
</script>
~~~

#### 路由懒加载(在调用该路由时才会加载组件，并进行缓存)

~~~js
const rout=()=>import("../components/Route2")
// 2. 定义路由配置
const routes = [
    {
        path:"/",
        component:rout
    }]
~~~

#### 状态管理

- 在src下新建文件store/index.js

~~~js
import {reactive} from "vue"
const store={
    msg:reactive({
        content:"helloworld"
    }),
    changeMsg:function () {
        this.msg.content="状态集中管理"
    }
}
export default store
~~~

- 在App.vue组件中使用组件并提供依赖

~~~html
<template>
    <div>
        <hello-world></hello-world>
    </div>
</template>
<script>
    import store from "./store/index"
    import helloWorld from "./components/HelloWorld"
    export default {
        provide:{           //提供依赖
            store
        },
        components:{
            helloWorld
        }
    }
</script>
~~~

- 子组件接收依赖并显示

~~~html
<template>
    {{store.msg.content}}
    <button @click="changeContect">改变集中管理内容</button>
</template>
<script>
    export default {
        name: "HelloWorld",
        inject:['store'],   //接收依赖
        methods:{
            changeContect:function () {
                this.store.changeMsg()
            }
        }
    }
</script>
~~~

#### 使用fetch获取数据

~~~js
created:function () {
    fetch("http://localhost:8081/Index/getMess").then((mess)=>{
    	return mess.json()
    }).then((message)=>{
    	this.Imgs=message.imgs
    })
}
~~~

#### 使用axios获取数据

> 需要加载axios依赖
>
> npm install axios

~~~js
import axios from "axios"
created:function () {
    let that=this;
    axios.post("http://localhost:8081/Index/getMess").then(function (mess) { //axios中获取不到this
        console.log(mess.data)
        that.Imgs=mess.data.imgs
    }).catch((error)=>{
    	console.log(error)
    })
}
~~~