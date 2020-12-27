<template>
  <div>
    <h3>setup</h3>
    <p class="ml15"> 1. setup 是在组件尚未创建时执行，所以在 setup 选项中并没有this</p>
    <p class="ml15"> 2. 除了 props 之外，将无法访问组件中声明的任何属性、本地状态、计算属性或者方法</p>
    <br>
    <pre>
      export default {
        setup() {
          // setup 是在组件尚未创建时执行，所以在 setup 选项中并没有this
          // 除了 props 之外，将无法访问组件中声明的任何属性、本地状态、计算属性或者方法
          console.log('setup', this)
          return {}
        },
        beforeCreate() {
          console.log('beforeCreate')
        },
        created() {
          console.log('created')
        }
      }
    </pre>
  </div>
</template>

<script>
  export default {
    setup() {
      // setup 是在组件尚未创建时执行，所以在 setup 选项中并没有this
      // 除了 props 之外，将无法访问组件中声明的任何属性、本地状态、计算属性或者方法
      console.log('setup', this)
      return {}
    },
    beforeCreate() {
      console.log('beforeCreate')
    },
    created() {
      console.log('created')
    }
  }
</script>

<style>

</style>