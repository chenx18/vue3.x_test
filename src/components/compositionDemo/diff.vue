vue3 performance（性能）提升分析
 1：diff算法优化
     vue2中的虚拟dom是进行全量的对比
                                vue3中新增了静态标记（patchFlag）,只对比带有PF的节点，并且通过Flag的信息得知当前节点要对比的具体内容
网址：https://vue-next-template-explorer.netlify.app

2：全局api分块(Tree Shaking机制)
   Tree Shaking称为“摇树优化” ；Tree Shaking就是把无用的模块进行“剪枝”，很多没有用到的Api就不会打包到最后的包里
    Tree Shaking的前提是所有的内容必须用 ES6 module的import


3：静态提升
                                vue2中无论元素是否参与更新，每次都会重新创建，然后再渲染
                                   vue3中对于不参与更新的元素，会做静态提升，只会被创建一次，在渲染时直接复用即可

4：cacheHandlers事件侦听器缓存
默认情况下onClick会被视为动态绑定，所以每次都会去追踪它的变化
但是因为是统一个函数，所以没有追踪变化，直接缓存起来复用就可以






