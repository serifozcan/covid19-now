<template>
  <div id="app">
    <Earth />
  <div class="top-layer">
      <Github />
  </div>
  </div>
</template>

<script>
import Earth from './components/Earth.vue'
import Github from './components/Github.vue'

export default {
  name: 'App',
  components: {
    Earth,
    Github
  }
}
</script>

<style>
html,
body {
  /* background-color: black; */
  background-color: #fcfcfa;
  color: white;
  margin: 0;
}
.top-layer {
  z-index: 10;
}
</style>
