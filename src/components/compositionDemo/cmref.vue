<template>
  <div>
    <h3>ref</h3>
    <p>ref 函数包装了一个响应式的数据对象，将值传递给对象的value 属性，每次访问都要加 .value </p>
    <p>在 template 模板中访问是需要加value，因此在编译时，会自动识别是否为ref包装过的</p>
    <br>
    <p>ref object: {{person}}</p>  
    <p>ref: {{person.count}}</p>  

    <pre>
      
      import {ref} from 'vue';
      export default {
        setup() {
          // ref 函数包装了一个响应式的数据对象，将值传递给对象的value 属性，每次访问都要加 .value 
          // 在 template 模板中访问是需要加value，因此在编译时，会自动识别是否为ref包装过的
          const person = ref({count: 3});
          console.log(person.vlaue)
          return {
            person
          }
        },
      }
    </pre>
  </div>
</template>

<script>
  import {ref} from 'vue';
  export default {
    setup() {
      // ref 函数包装了一个响应式的数据对象，将值传递给对象的value 属性，每次访问都要加 .value 
      // 在 template 模板中访问是需要加value，因此在编译时，会自动识别是否为ref包装过的
      const person = ref({count: 3});
      // console.log(person.vlaue)
      return {
        person
      }
    },
  }
</script>

<style>

</style>