<template>
  <div>
    <h3>reactive</h3>
    <p class="ml15"> 1. reactive 函数用于创建一个响应式的数据对象，很好的解决了vue2 通过 defineProperty 实现数据响应式的缺陷</p>
    <p class="ml15"> 2. 只需要把数据作为参数传入就可以,不需要 .value 访问，；</p>
    <br>
    <p class="ml15">name: {{person.name}}</p>  
    <p class="ml15">age: {{person.age}}</p>  

    <br>
    <br>
    <pre>
      import {reactive} from 'vue';
      export default {
        setup() {
          // reactive 函数用于创建一个响应式的数据对象，很好的解决了vue2 通过 defineProperty 实现数据响应式的缺陷
          // 不需要 .value 就可访问，只需要把数据作为参数传入就可以；
          const person = reactive({
            name:'Tony',
            age: 18
            });
          console.log('reactive',person)
          // 将响应式数据对象 state return 出去，供template 使用
          return {
            person
          }
        },
      }
    </pre>
    <br>

    <h3>如何选择 ref 和 reactive</h3>
    <p class="ml15"> 1. 基本数据类型（string、number、boolean）或者单值对象（{count: 123}）只有一个属性值的对象使用ref </p>
    <p class="ml15"> 2. 引用类型的值 （object、 Array）复杂的数据类型是用reactive</p>


  </div>
</template>

<script>
  import {reactive} from 'vue';
  export default {
    setup() {
      // reactive 函数用于创建一个响应式的数据对象，很好的解决了vue2 通过 defineProperty 实现数据响应式的缺陷
      // 不需要 .value 就可访问，只需要把数据作为参数传入就可以；
      const person = reactive({
        name:'Tony',
        age: 18
        });
      console.log('reactive',person)
      // 将响应式数据对象 state return 出去，供template 使用
      return {
        person
      }
    },
  }
</script>

<style>

</style>