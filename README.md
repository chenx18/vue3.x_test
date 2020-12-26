# vue3.x-admin 

### 前言
  这是一个完全使用 vue3.x 语法开发的后台管理系统
  页面主要包括 CSS3特效， 前端解析xlsx，前端文件下载，可拖拽的div，图表，益智小游戏，vuex4.x 存储用户信息等功能，
  该项目基本覆盖了vue3.x全家桶的方方面面。 
  功能丰富，注释多多， 近期会更新D3.js图表，并会进行一些代码优化，欢迎大家提意见。
  

### demo: [传送阵](http://114.215.147.221:8086/login)
### 技术栈
  ```js
    Vue3.x + vue-router4.x + vuex4.x + Less + Echarts
  ```

  ### 项目运行
  ```js
  git clone https://github.com/Tyf2345/vue3.x-admin.git
  cd vue-admin-ele
  yarn install
  yarn serve
  ```

  ### vue2.x版本传送地址 (https://github.com/Tyf2345/vue-admin-ele)


### 说明

>  如果对您有帮助，您可以点右上角 "Star" 支持一下 谢谢！ ^_^

>  或者您可以 "follow" 一下，我会不断开源更多的有趣的项目

>  如有问题请直接在 Issues 中提

### 目标功能
- [x] 登录/登出 -- 完成
- [x] 首页 -- 完成
- [x] icon -- 完成
- [x] table -- 完成
- [x] 柱图 -- 完成
- [x] 折线图 -- 完成
- [x] 拖拽div -- 完成
- [x] 前端解析xlsx -- 完成
- [x] 个贪吃蛇小游戏 -- 完成
- [x] 人中心 -- 完成

### 结构目录

├── build                                     // webpack配置文件 <br/>
├── config                                      // 项目打包路径 <br/>
├── src                                         // 源码目录 <br/>
│   ├── assets                                  // 公共资源， 公共css <br/>
│   ├── components                              // 公共组件 <br/>
│   ├── docs                                    // 前端上传 解析xlsx文件 <br/>
│   ├── mock                                    // 异步模拟ajax调用接口 <br/>
│   ├── router                                  // 路由相关 <br/>
│   ├── vuex                                    // 状态管理相关 <br/>
│   ├── utils                                   // 工具类，方法 <br/>
│   ├── views                                   // 页面相关 <br/>
│   ├── App.vue                                 // 页面入口文件 <br/>
│   ├── main.js                                 // 程序入口文件，加载各种公共组件 <br/>
├── index.html                                  // 入口html文件 <br/>

### vue 3.x 学习路线

- Vue.js 3.0 的优化  
- Vue.js 核心组件的实现（上）
- Vue 核心的实现（下）

01 | 组件渲染：vnode 到真实 DOM 是如何转变的？
02 | 组件更新：完整的 DOM diff 流程是怎样的？（上）
03 | 组件更新：完整的 DOM diff 流程是怎样的？（下）

模块二：学会新设计 Composition API
模块二导读 | 逻辑复用最佳实践：Composition API
04 | Setup：组件渲染前的初始化过程是怎样的？
05 | 响应式：响应式内部的实现原理是怎样的？（上）
06 | 响应式：响应式内部的实现原理是怎样的？（下）
07 | 计算属性：计算属性比普通函数好在哪里？
08 | 侦听器：侦听器的实现原理和使用场景是什么？（上）
09 | 侦听器：侦听器的实现原理和使用场景是什么？（下）
10 | 生命周期：各个生命周期的执行时机和应用场景是怎样的？
11 | 依赖注入：子孙组件如何共享数据？
 
模块三：编译过程和背后的优化思想
12 | 模板解析：构造 AST 的完整流程是怎样的？
13 | AST 转换：AST 节点内部做了哪些转换？
14 | 生成代码：AST 如何生成可运行的代码？

模块四：探索更多实用特性背后的实现原理
15 | Props：Props 的初始化和更新流程是怎样的？
16 | 插槽：如何实现内容分发？
17 | 指令：指令完整的生命周期是怎样的？
18 | v-model：双向绑定到底是怎么实现的？

模块五：学习 Vue 内置组件的实现原理
19 | Teleport 组件：如何脱离当前组件渲染子组件？
20 | Suspence 组件：如何优雅地实现组件异步处理流程？
21 | KeepAlive 组件：如何让组件在内存中缓存和调度？
22 | Transition 组件：过渡动画的实现原理是怎样的？
22 | Vue 官方生态的实现原理
23 | Vue Router：如何实现一个前端路由？
24 | Vuex：如何实现前端的状态管理？