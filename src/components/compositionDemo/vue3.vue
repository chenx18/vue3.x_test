vue3概述
https://composition-api.vuejs.org/zh/api.html#setup
                官网：https://vue3js.cn/
                                下一代web开发方式，更快，更轻，易维护，更多的原生支持
      vue3是完全兼容vue2的
vue3特性
     1：创建实例的方式
                                                vue2以new Vue（{}）.$mount('#app')
                                                                                vue3 以Vue.createApp().mount('#app')
                                2:Teleport 
                                                                                将一个元素或者一个组件渲染到规定的节点中，可以脱离vue范围
                                3：自定义渲染函数
                        4：响应性原理改变
                                                vue2中用的是Object.defineProperty的getter和setter
                                                                                                                                上面的方法实现拦截【手动设置】
                                                                                vue3采用的是全新的es6语法 proxy【除了ie之外的其他浏览器】
                       底层采用了proxy的effect和trigger
                                                                                ie浏览器依旧采用的是Object.defineProperty
      5：vue3新增了全新一套功能  --组合式API--Composition API
         这套功能你可以用也可以不用
     6：Fragment模板实现多个根元素

vue2升级到vue3
     先把教授级升级到最新版本，如果之前没有安装过，直接全局安装最新版本
      npm install -g @vue/cli

    如果已经安装过，可以升级版本

			
npm update -g @vue/cli
or

yarn global  upgrade --latest @vue/cli

			
  创建vue3项目
			

			

				
vue create vue3  //默认选择vue3
cd vue3
yarn serve

				
 如果忘记选择vue3，你创建的项目默认选择的是v2.0 你执行以下操作
				

			

			


			
 yarn add vue-next
or
add vue-next


