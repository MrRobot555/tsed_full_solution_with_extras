<template>
  <div id="app">
    <div class="container">
      <tsedform />
    </div>
  </div>
</template>

<script>
import tsedform from './components/TsedForm.vue'
export default {
  name: 'App',
  components: {
    tsedform
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
