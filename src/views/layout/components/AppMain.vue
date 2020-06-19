<template>
  <section class="app-main">
    <transition name="fade-transform" mode="out-in">
      <!-- or name="fade" -->
      <!-- 如果定一个唯一id，那么强制重新渲染 -->
      <router-view :key="key"/>
      <!-- 如果路由出口指向的页面组件是同一个，那么出口显示的页面组件不会被重新暄软 -->
      <router-view/>
    </transition>
  </section>
</template>

<script>
export default {
  name: 'AppMain',
  // 配合这里，计算属性
  computed: {
    key() {
      return this.$route.name !== undefined ? this.$route.name + +new Date() : this.$route + +new Date()
    }
  }
}
</script>

<style scoped>
.app-main {
  /*50 = navbar  */
  min-height: calc(100vh - 50px);
  position: relative;
  overflow: hidden;
}
</style>
