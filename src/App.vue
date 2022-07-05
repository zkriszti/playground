<template>
  <div id="app">
    <nav>
      <ul>
        <li :class="regularActive ? 'is-active' : null" @click="regularActive=true">Regular input version</li>
        <li :class="regularActive ? null : 'is-active'" @click="regularActive=false">String input version</li>
      </ul>
    </nav>
    <TestDisplay v-if="regularActive" />
    <StringInputDisplay v-else />
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
  },
  data () {
    return {
      regularActive: true
    }
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

nav ul {
  list-style-type: none
}

nav ul li {
  display: inline-block;
  margin-right: 24px;
  cursor: pointer
}

nav ul li.is-active {
  border-bottom: 2px solid blue
}
</style>
