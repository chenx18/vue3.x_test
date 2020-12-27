<template>
  <div>
    <h3>toRefs</h3>
    <p class="ml15">1. toRefs 将传入的对象里所有的属性的值转化为响应式数据，该函数支持一个参数，即obj对象</p>
    <br>
    <!-- 绑定的是obj对象数据，return时需解构 -->
    <p class="ml15">未解构torefs -- {{state}}</p>

    <br>
    <p class="ml15">姓名: {{name}}</p>
    <p class="ml15">年龄: {{age}}</p>
    <p class="ml15">职业: {{work}}</p>

    <br>
    <pre>
      import {toRefs} from 'vue';
      export default {
        setup(){
          const obj = {
            name: 'Tony',
            age: 18,
            work: 'lifa'
          }
          // 将 obj 对象的所有属性转化为响应式数据
          const state= toRefs(obj);
          console.log(state);
          // 返回一个对象，对象里包含了每一个包装过的响应式数据对象
          return {
            state,
            ...state
          }
        }
      }
    </pre>



  </div>
</template>

<script>
import {toRefs} from 'vue';
export default {
  setup(){
    const obj = {
      name: 'Tony',
      age: 18,
      work: 'lifa'
    }
    // 将 obj 对象的所有属性转化为响应式数据
    const state= toRefs(obj);
    console.log(state);
    // 返回一个对象，对象里包含了每一个包装过的响应式数据对象
    return {
      state,
      ...state
    }
  }
}
</script>

<style>

</style>