<template>
  <div id="app">
    <div class="nav-bar">
      <button 
        @click="currentView = 'exam'" 
        :class="{ active: currentView === 'exam' }"
      >
        考试
      </button>
      <button 
        @click="currentView = 'scores'" 
        :class="{ active: currentView === 'scores' }"
      >
        成绩查询
      </button>
    </div>
    
    <div v-if="currentView === 'exam'">
      <button @click="goToScores" class="test-btn">测试按钮 - 查看考生成绩</button>
      <SurveyExam />
    </div>
    <ScoreList v-if="currentView === 'scores'" />
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
      currentView: 'exam'
    }
  },
  methods: {
    goToScores() {
      this.currentView = 'scores'
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

.nav-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
  background-color: white;
  border-radius: 8px;
  padding: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.nav-bar button {
  background: none;
  border: none;
  padding: 10px 20px;
  margin: 0 5px;
  font-size: 16px;
  color: #555;
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.nav-bar button:hover {
  background-color: #f5f5f5;
  color: #42b983;
}

.nav-bar button.active {
  background-color: #42b983;
  color: white;
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
