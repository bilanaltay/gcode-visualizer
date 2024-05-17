<template>
 <div id="app">
    <GCodeVisualizer />
  </div>
</template>

<script>
import GCodeVisualizer from './components/GCodeVisualizer.vue'

export default {
  name: 'App',
  components: {
    GCodeVisualizer
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
