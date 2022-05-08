<template>
  <div id="app" class="col-12 col-sm-10 col-lg-6" style="margin: 0 auto">
    <OTCTool />
  </div>
</template>

<script>
import OTCTool from './components/OTCTool.vue';

export default {
  name: 'App',
  components: {
    OTCTool,
  },
};
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
