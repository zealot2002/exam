<template>
  <div id="app">
    <div v-if="showScores">
      <ScoreList />
    </div>
    <div v-else>
      <SurveyExam @backdoor-triggered="showScoresList" />
    </div>
  </div>
</template>

<script>
import SurveyExam from './components/SurveyExam.vue'
import ScoreList from './components/ScoreList.vue'

export default {
  name: 'App',
  components: {
    SurveyExam,
    ScoreList
  },
  data() {
    return {
      showScores: false
    }
  },
  methods: {
    showScoresList() {
      this.showScores = true
    }
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

.test-btn {
  position: fixed;
  top: 10px;
  right: 10px;
  background-color: #f39c12;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 16px;
  font-size: 14px;
  cursor: pointer;
  z-index: 100;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.test-btn:hover {
  background-color: #e67e22;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
