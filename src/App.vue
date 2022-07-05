<template>
  <div id="app">
    <TestDisplay />
    <!-- <StringInputDisplay /> -->
  </div>
</template>

<script>
import TestDisplay from './components/TestDisplay.vue'
import StringInputDisplay from './components/StringInputDisplay.vue'

export default {
  name: 'App',
  components: {
    TestDisplay,
    StringInputDisplay    
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
