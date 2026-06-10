<template>
  <Header />
  <div class="layout-main">
    <Aside />
    <router-view class="content-view" />
  </div>
</template>

<script>
import Header from '../components/Header'
import Aside from '../components/Aside'

export default {
  name: 'Layout',
  components: { Header, Aside }
}
</script>

<style scoped>
.layout-main { display: flex; min-height: calc(100vh - 64px); background: #f5f7fb; }
.content-view { flex: 1; overflow: auto; }
</style>