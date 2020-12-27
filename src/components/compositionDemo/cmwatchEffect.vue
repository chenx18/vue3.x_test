<template>
  <div>
    <h3>watch</h3>
    <p class="ml15">
      watch 和 watchEffect 都是用来监听某项数据变化而执行操作的，两者的用法有所区别
    </p>
    <br>
    
    <h3>watchEffect</h3>
    <p class="ml15">1. 不需要手动传入依赖</p>
    <p class="ml15">2. 每次初始化时会执行一次回调函数来自动获取依赖</p>
    <p class="ml15">3. 无法获取到原值，只能到变化后的值</p>
  </div>
</template>

<script>
import {reactive, ref, watchEffect} from 'vue';
export default {
  setup(){

    // watchEffect 会组件初始化时，将该回调函数执行一次，自动获取到需要检查的数据
    const state3 = reactive({name: 'Tony', count: 0});
    watchEffect(() =>{
        console.log(`state3 name原始值 ${oldName}`);
        console.log(`state3 name新值 ${newName}`);

        console.log(`state3 count原始值 ${oldCount}`);
        console.log(`state3 count新值： ${newCount} `);
      }
    )
    setTimeout(() => {
      state3.count++
      state3.name="zhang"
    }, 2000);

  }
}
</script>

<style>

</style>