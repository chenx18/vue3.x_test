<template>
  <div>
    <h3>toref</h3>
    <p class="ml15">1. toref 将某个对象的值转化为响应式数据，其接收两个参数，第一个参数是obj 数据， 第二个参数为对象中的属性名</p>
    <br>
    <p class="ml15">toref = {{state}}</p>

    <br>
    <pre>
      import {toRef} from 'vue';
      export default {
        setup(){
          const obj = {count: 3}
          // 将 obj 对象中的属性 count 转化为响应式数据
          const state= toRef(obj,'count');
          // 将包装过的数据对象返回给template使用
          return {
            state
          }
        }
      }
    </pre>

    <h3>ref 与toRef 区别</h3>
    <p class="ml15">1. ref是对传入数据的拷贝；toRef是对传入数据的引用</p>
    <p class="ml15">2. ref的值改变会更新视图；toRef的值改变不会更新视图</p>
    <br>
    <p class="ml15 mb15"> {{state1}}  <button @click="add1">增加ref</button></p>
    <p class="ml15"> {{state2}}  <button @click="add2">增加toRef</button></p>
    <br>
    <pre>
      import {ref,toRef} from 'vue';
      export default {
        setup(){
          const obj1={count: 1}
          // ref
          // 视图改变 原始值不变 响应数据对象改变
          // ref是对原始数据的一个拷贝，不会影响到原始值
          // 响应式数据对象值的改变会更新到视图
          const state1 = ref(obj1.count);
          function add1() {
            state1.value++;
            console.log('原始值=', obj1);
            console.log('ref 响应式数据对象=', state1)
          }

          // toref
          // 视图不变 原始值改变 响应数据对象改变
          // toRef是对原数据的一个引用会影响到原始值
          // 响应式数据对象值改变不会更新视图
          const state2 = toRef(obj1, 'count');
          function add2() {
            state2.value++;
            console.log('原始值=', obj1);
            console.log('ref 响应式数据对象=', state2)
          }

          console.log()

          return {
            state1,
            state2,
            add1,
            add2
          }
        }
      }
    </pre>


  </div>
</template>

<script>
import {ref,toRef} from 'vue';
export default {
  setup(){
    // -------------toref--------
    const obj = {count: 3}
    // 将 obj 对象中的属性 count 转化为响应式数据
    const state= toRef(obj,'count');
    

    // -------------toref ref 对比--------
    const obj1={count: 0}

    // ref
    // 视图改变 原始值不变 响应数据对象改变
    // ref是对原始数据的一个拷贝，不会影响到原始值
    // 响应式数据对象值的改变会更新到视图
    const state1 = ref(obj1.count);
    function add1() {
      state1.value++;
      console.log('原始值=', obj1);
      console.log('ref 响应式数据对象=', state1)
    }

    // toref
    // 视图不变 原始值改变 响应数据对象改变
    // toRef是对原数据的一个引用会影响到原始值
    // 响应式数据对象值改变不会更新视图
    const state2 = toRef(obj1, 'count');
    function add2() {
      state2.value++;
      console.log('原始值=', obj1);
      console.log('ref 响应式数据对象=', state2)
    }

    console.log()

    return {
      state,
      state1,
      state2,
      add1,
      add2
    }
  }
}
</script>

<style>

</style>