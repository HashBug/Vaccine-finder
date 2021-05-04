<template>
  <div id="app">
    <VaccineFinder url="https://cdn-api.co-vin.in/api/v2" />
  </div>
</template>

<script>
import VaccineFinder from "./components/VaccineFinder.vue";

export default {
  name: "App",
  components: {
    VaccineFinder,
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
