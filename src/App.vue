<template>
  <div id="app">
    <topic-tree/>
  </div>
</template>

<script>
import TopicTree from './components/tree.vue'

export default {
  name: 'app',
  components: {TopicTree}
  // data () {
  //   return data
  // }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
