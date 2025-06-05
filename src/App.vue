<template>
  <div id="app">
    <SurveyExam />
  </div>
</template>

<script>
import SurveyExam from './components/SurveyExam.vue'

export default {
  name: 'App',
  components: {
    SurveyExam
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
  margin-top: 30px;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
  padding: 0 20px;
}

body {
  background-color: #f5f5f5;
  margin: 0;
  padding: 0;
}

h1 {
  margin-bottom: 30px;
  color: #42b983;
}
</style>
