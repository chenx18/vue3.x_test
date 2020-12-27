<template>
  <div>
    <h3>watch</h3>
    <p class="ml15">
      watch 和 watchEffect 都是用来监听某项数据变化而执行字第操作的，两者的用法有所区别
    </p>
    <br>
    
    <h3>watch(source,cb, [options])</h3>
    <p class="ml15">source: 可以是表达式或函数，用于指定监听的依赖对象</p>
    <p class="ml15">cb: 依赖对象变化后执行的回调函数</p>
    <p class="ml15">options: 可选参数， 可以配置的属性有 immediate（立即触发回调函数），deep(深度监听)</p>
    

    <h3>watch 监听ref数据</h3>
    <pre>
      const state1 = ref(0);
      watch(state1,(newval,oldval) => {
        console.log('原始值：', oldval);
        console.log('新值：', newval);
      })
      setTimeout(() => {
        state1.value++
      }, 2000);
    </pre>
    <br>
    <h3>监听 reactive 数据</h3>
    <pre>
      const state2 = reactive({count: 0});
      watch(() => state2.count,(newval, oldval) => {
        console.log(`state2原始值 ${oldval}`);
        console.log(`state2新值 ${newval}`);
      })
      setTimeout(() => {
        state2.count++
      }, 2000);
    </pre>

    <br>
    <h3>监听多个属性</h3>
    <pre>
      const state3 = reactive({name: 'Tony', count: 0});
      watch(
        [()=>state3.name, ()=>state3.count],
        ([newName, newCount], [oldName, oldCount]) =>{
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
    </pre>

    <h3>初始化组件时就执行回调函数 immediate: true</h3>
    <pre>
      const state4 = reactive({name: '李四', count: 10});
      watch(
        [()=>state4.name, ()=>state4.count],
        ([newName, newCount], [oldName, oldCount]) =>{
          console.log(`state4 name原始值 ${oldName}`);
          console.log(`state4 name新值 ${newName}`);

          console.log(`state4 count原始值 ${oldCount}`);
          console.log(`state4 count新值： ${newCount} `);
        },
        {
          immediate: true
        }
      )
      setTimeout(() => {
        state4.count++
        state4.name="LI"
      }, 2000);
    </pre>

    <h3>监听多层嵌套，设置 deep:true;</h3>
    <p><button @click="stop">停止监听state6</button></p>
    <pre>
      const state5 = reactive({obj: {name: '李四'}})
      watch(() => state5.obj.name,(newval, oldval) => {
        console.log(`state5原始值 ${oldval}`);
        console.log(`state5新值 ${newval}`);
      },{deep: true})
      setTimeout(() => {
        state5.obj.name='zhang'
      }, 2000);

      // watch 会返回一个stop方法，如果想要停止，可以执行该stop函数
      const state6 = reactive({obj: {name: '彭于晏'}})
      const stop = watch(() => state6.obj.name,(newval, oldval) => {
        console.log(`state6原始值 ${oldval}`);
        console.log(`state6新值 ${newval}`);
      },{deep: true})
      setTimeout(() => {
        state6.obj.name='peng'
      }, 2000);

      return {
        stop
      }
    </pre>
  </div>
</template>

<script>
import {reactive, ref, watch} from 'vue';
export default {
  setup(){
    // watch 监听ref数据
    const state1 = ref(0);
    watch(state1,(newval,oldval) => {
      console.log('原始值：', oldval);
      console.log('新值：', newval);
    })
    setTimeout(() => {
      state1.value++
    }, 2000);

    // 监听 reactive 数据
    const state2 = reactive({count: 0});
    watch(() => state2.count,(newval, oldval) => {
      console.log(`state2原始值 ${oldval}`);
      console.log(`state2新值 ${newval}`);
    })
    setTimeout(() => {
      state2.count++
    }, 2000);

    // 监听多个属性
    const state3 = reactive({name: 'Tony', count: 0});
    watch(
      [()=>state3.name, ()=>state3.count],
      ([newName, newCount], [oldName, oldCount]) =>{
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

    // 初始化组件时就执行回调函数 immediate: true
    const state4 = reactive({name: '李四', count: 10});
    watch(
      [()=>state4.name, ()=>state4.count],
      ([newName, newCount], [oldName, oldCount]) =>{
        console.log(`state4 name原始值 ${oldName}`);
        console.log(`state4 name新值 ${newName}`);

        console.log(`state4 count原始值 ${oldCount}`);
        console.log(`state4 count新值： ${newCount} `);
      },
      {
        immediate: true
      }
    )
    setTimeout(() => {
      state4.count++
      state4.name="LI"
    }, 2000);


    // 监听多层嵌套，设置 deep:true;
    const state5 = reactive({obj: {name: '李四'}})
    watch(() => state5.obj.name,(newval, oldval) => {
      console.log(`state5原始值 ${oldval}`);
      console.log(`state5新值 ${newval}`);
    },{deep: true})
    setTimeout(() => {
      state5.obj.name='zhang'
    }, 2000);

    // watch 会返回一个stop方法，如果想要停止，可以执行该stop函数
    const state6 = reactive({obj: {name: '彭于晏'}})
    const stop = watch(() => state6.obj.name,(newval, oldval) => {
      console.log(`state6原始值 ${oldval}`);
      console.log(`state6新值 ${newval}`);
    },{deep: true})
    setTimeout(() => {
      state6.obj.name='peng'
    }, 2000);

    return {
      stop
    }
  }
}
</script>

<style>

</style>