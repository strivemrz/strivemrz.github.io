---
title: vue-cli脚手架安装
categories: vue
tags:
  - vue

excerpt: ''

cover: https://ts1.cn.mm.bing.net/th/id/R-C.21d4b5812b720fcef2204ff11c664772?rik=mO2Wz6ZL%2bllB5w&riu=http%3a%2f%2fup.deskcity.org%2fpic%2f16%2f33%2fdf%2f1633df034b4cb7a8c46666683b3a6ffb.jpg&ehk=VGapTpgOufJok9HUN8v%2beu65Vuoka70rZsXSMs%2b3haM%3d&risl=&pid=ImgRaw&r=0

---



#### Lint工具

> 提供编码规范；
>
> 提供自动检验代码的程序，并打印检验结果：告诉你哪一个文件哪一行代码不符合哪一条编码规范，方便你去修改代码。
>
> Lint是检验代码格式工具的一个统称，具体的工具有Jslint,Eslint等等。

#### 安装vue脚手架

##### 安装node.js(包含npm)

1. 安装cnpm

> 由于有些npm资源被屏蔽或者是国外资源的原因，经常会导致npm安装依赖包的时候失败，所以我们还需要npm的国内镜像----cnpm.

~~~bash
npm install -g cnpm --registry=http://registry.npm.taobao.org
~~~

2. 安装vue脚手架/卸载vue脚手架

~~~bash
npm install -g vue-cli #2.x版本
npm install -g @vue/cli #3.0版本以上

#卸载vue脚手架
npm uninstall -g vue-cli #卸载2.x版本
npm uninstall -g @vue/cli #卸载3.x版本以上的

npm list vue -g #查看vue版本
~~~

3. 查看是否安装成功

~~~bash
vue -V
~~~

4. 创建vue项目

~~~bash
vue init webpack firshApp（初始化一个完整的项目）
# 其中webpack是构建工具，也就是整个项目是基于webpack的。其中firstApp是整个项目文件夹的名称，这个文件夹会自动生成在你指定的目录中
~~~

- Project name :项目名称 ，如果不需要更改直接回车就可以了。注意：这里不能使用大写，所以我把名称改成了vueclient
- Project description:项目描述，默认为A Vue.js project,直接回车，不用编写。
- Author：作者，如果你有配置git的作者，他会读取。
- Install  vue-router? 是否安装vue的路由插件，我们这里需要安装，所以选择Y
- Use ESLint to lint your code? 是否用ESLint来限制你的代码错误和风格。我们这里不需要输入n（建议），如果你是大型团队开发，最好是进行配置。
- setup unit tests with  Karma + Mocha? 是否需要安装单元测试工具Karma+Mocha，我们这里不需要，所以输入n。
- Setup e2e tests with Nightwatch?是否安装e2e来进行用户行为模拟测试，我们这里不需要，所以输入n

5. 各目录作用

```text
 build：最终发布的代码的存放位置。
 
 config：配置路径、端口号等一些信息，我们刚开始学习的时候选择默认配置。

 node_modules：npm 加载的项目所需要的各种依赖模块。

 src：这里是我们开发的主要目录（源码），基本上要做的事情都在这个目录里面，里面包含了几个目录及文件：

 assets:放置一些图片(会根据图片大小分类进行base64命名还是其他方式命名)，如logo等

 components:目录里放的是一个个的组件文件

 router/index.js：配置路由的地方

 App.vue：项目入口组件（根组件），我们也可以将组件写这里，而不使用components目录。主要作用就是将我们自己定义的组件通过它与页面建立联系进行渲染，这里面的<router-view/>必不可少。

 main.js ：项目的核心文件（整个项目的入口js）引入依赖包、默认页面样式等（项目运行后会在index.html中形成一个app.js文件）。

 static：静态资源目录(会原分不动的对文件进行处理)，如图片、字体等。

 test：初始测试目录，可删除

 .XXXX文件：配置文件。

 index.html：html单页面的入口页面，可以添加一些meta信息或者同统计代码啥的或页面的重置样式等。

 package.json：项目配置信息文件/所依赖的开发包的版本信息及所依赖的插件信息。（大概版本）

 package-lock.json：项目配置信息文件/所依赖的开发包的版本信息及所依赖的插件信息。（具体版本）

 README.md：项目的说明文件。

 webpack.config.js：webpack的配置文件，例：把.vue的文件打包成浏览器能读懂的文件。

 .babelrc:是检测es6语法的配置文件，例：适配哪些浏览器的限制

 .gitignore:上传到服务器忽略哪些文件的配置（比如模拟本地数据mock不让他在get提交/打包上线的时候忽略不使用可在这里配置）

 .postcssrc.js:前缀的配置 （css转化的配置）

 .editorconfig:对代码进行规范，例：root是否进行检测，代码尾部是否换行，缩行前面几个空格...（建议定义这个规范）

 .eslintrc.js:配置eslint语法规则（在这里面的rules属性中配置让哪个语法规则失效）

 .eslintignore:忽略eslint对项目某些文件的语法规则的检查
```
6. 安装项目所需要的依赖包/插件(如果前一步没有报错，则此步可以省略)

 cd  项目名；进入项目中

```text
安装项目所需要的依赖包/插件（在package.json可查看）：执行 cnpm install   (npm可能会有警告，这里可以用cnpm代替npm了，运行别人的代码需要先安装依赖)如果创建项目的时候没有报错，这一步可以省略。如果报错了  cd到项目里面运行  cnpm install   /  npm install
```
**若拿到别人的项目或从github上下载的项目第一步就是要在项目中cnpm install;下载项目所依赖的插件，然后npm run dev 运行项目**



安装完成之后项目会多一个node_modules文件夹，这里面就是我们所需要的依赖包资源

7. 运行项目

npm run dev / npm run start

会用热加载的方式运行项目，修改完代码后，不用手动刷新浏览器就能看到修改后的效果

8. 打包命令

cnpm run build 

会生成一个dist文件，就是打包文件，点击.html能运行则成功