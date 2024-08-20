<template>
  <AtexBuilder />
</template>

<script>
import AtexBuilder from './components/AtexBuilder.vue'

export default {
  name: 'App',
  components: {
    AtexBuilder
  }
}
</script>

<style>
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
