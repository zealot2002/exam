<template>
  <div class="score-list">
    <h1>考试成绩列表</h1>
    
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <div v-else-if="error" class="error">
      {{ error }}
    </div>
    
    <div v-else>
      <div class="search-filter">
        <input 
          type="text" 
          v-model="searchTerm" 
          placeholder="搜索考生姓名、手机号或组织..."
          @input="filterScores"
        >
      </div>

      <table class="scores-table">
        <thead>
          <tr>
            <th>考生姓名</th>
            <th>手机号码</th>
            <th>所属组织</th>
            <th>得分</th>
            <th>总题数</th>
            <th>提交时间</th>
            <th>操作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="score in filteredScores" :key="score.id">
            <td>{{ score.student_name }}</td>
            <td>{{ score.phone }}</td>
            <td>{{ score.organization }}</td>
            <td :class="getScoreClass(score.score, score.total_questions)">
              {{ score.score }}/{{ score.total_questions * 5 }}
            </td>
            <td>{{ score.total_questions }}</td>
            <td>{{ score.submission_time }}</td>
            <td>
              <button @click="showDetails(score)" class="detail-btn">查看详情</button>
            </td>
          </tr>
        </tbody>
      </table>
      
      <div class="pagination" v-if="totalPages > 1">
        <button 
          :disabled="currentPage === 1" 
          @click="changePage(currentPage - 1)"
        >
          上一页
        </button>
        <span>{{ currentPage }} / {{ totalPages }}</span>
        <button 
          :disabled="currentPage === totalPages" 
          @click="changePage(currentPage + 1)"
        >
          下一页
        </button>
      </div>
    </div>
    
    <!-- 详情弹窗 -->
    <div class="modal" v-if="selectedScore">
      <div class="modal-content">
        <div class="modal-header">
          <h2>{{ selectedScore.student_name }} 的考试详情</h2>
          <button class="close-btn" @click="selectedScore = null">&times;</button>
        </div>
        <div class="modal-body">
          <div class="student-info">
            <p><strong>姓名:</strong> {{ selectedScore.student_name }}</p>
            <p><strong>手机号:</strong> {{ selectedScore.phone }}</p>
            <p><strong>所属组织:</strong> {{ selectedScore.organization }}</p>
            <p><strong>得分:</strong> {{ selectedScore.score }}/{{ selectedScore.total_questions * 5 }}</p>
            <p><strong>提交时间:</strong> {{ selectedScore.submission_time }}</p>
            <p v-if="selectedScore.remarks"><strong>备注:</strong> {{ selectedScore.remarks }}</p>
          </div>
          
          <h3>答题详情</h3>
          <div class="answers-list">
            <div 
              v-for="(answer, index) in selectedScore.answers" 
              :key="answer.question_id"
              class="answer-item"
              :class="{ correct: answer.is_correct, incorrect: !answer.is_correct }"
            >
              <div class="question-header">
                <span class="question-number">问题 {{ index + 1 }}</span>
                <span class="question-status" :class="{ correct: answer.is_correct, incorrect: !answer.is_correct }">
                  {{ answer.is_correct ? '正确' : '错误' }}
                </span>
              </div>
              <div class="question-text">{{ answer.question_text }}</div>
              <div class="answer-details">
                <p><strong>所选答案:</strong> {{ answer.selected_answer }}</p>
                <p><strong>正确答案:</strong> {{ answer.correct_answer }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ScoreList',
  data() {
    return {
      scores: [],
      filteredScores: [],
      loading: true,
      error: null,
      selectedScore: null,
      searchTerm: '',
      currentPage: 1,
      pageSize: 10,
      total: 0,
      apiBaseUrl: '/api'
    }
  },
  computed: {
    totalPages() {
      return Math.ceil(this.total / this.pageSize)
    }
  },
  mounted() {
    this.fetchScores()
  },
  methods: {
    async fetchScores() {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch(`${this.apiBaseUrl}/scores`)
        const data = await response.json()
        
        if (data.success) {
          this.scores = data.scores
          this.filteredScores = [...this.scores]
          this.total = data.total
        } else {
          this.error = '获取成绩数据失败'
        }
      } catch (error) {
        this.error = `获取成绩数据时出错: ${error.message}`
      } finally {
        this.loading = false
      }
    },
    
    filterScores() {
      if (!this.searchTerm.trim()) {
        this.filteredScores = [...this.scores]
        return
      }
      
      const term = this.searchTerm.toLowerCase().trim()
      this.filteredScores = this.scores.filter(score => 
        score.student_name.toLowerCase().includes(term) ||
        score.phone.toLowerCase().includes(term) ||
        (score.organization && score.organization.toLowerCase().includes(term))
      )
    },
    
    showDetails(score) {
      this.selectedScore = score
    },
    
    changePage(page) {
      if (page < 1 || page > this.totalPages) return
      this.currentPage = page
      // If pagination is handled by the server, fetch data for the new page
      // this.fetchScores(page)
    },
    
    getScoreClass(score, totalQuestions) {
      const percentage = (score / (totalQuestions * 5)) * 100
      if (percentage >= 90) return 'excellent'
      if (percentage >= 70) return 'good'
      if (percentage >= 60) return 'pass'
      return 'fail'
    }
  }
}
</script>

<style scoped>
.score-list {
  text-align: left;
}

.loading, .error {
  padding: 20px;
  text-align: center;
}

.error {
  color: #e74c3c;
}

.search-filter {
  margin-bottom: 20px;
}

.search-filter input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.scores-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.scores-table th, .scores-table td {
  padding: 12px 15px;
  border-bottom: 1px solid #ddd;
  text-align: left;
}

.scores-table th {
  background-color: #f8f9fa;
  font-weight: bold;
}

.scores-table tr:hover {
  background-color: #f1f1f1;
}

.detail-btn {
  background-color: #42b983;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.detail-btn:hover {
  background-color: #3aa876;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin-top: 20px;
}

.pagination button {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background-color: white;
  cursor: pointer;
  border-radius: 4px;
}

.pagination button:disabled {
  background-color: #f5f5f5;
  cursor: not-allowed;
  color: #aaa;
}

/* Score coloring */
.excellent {
  color: #27ae60;
  font-weight: bold;
}

.good {
  color: #2980b9;
  font-weight: bold;
}

.pass {
  color: #f39c12;
}

.fail {
  color: #e74c3c;
}

/* Modal styling */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  border-radius: 8px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-bottom: 1px solid #ddd;
}

.modal-header h2 {
  margin: 0;
  font-size: 20px;
  color: #333;
}

.close-btn {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  color: #666;
}

.modal-body {
  padding: 20px;
}

.student-info {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 6px;
  margin-bottom: 20px;
}

.student-info p {
  margin: 8px 0;
}

.answers-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.answer-item {
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 15px;
  background-color: #fff;
}

.answer-item.correct {
  border-left: 4px solid #27ae60;
}

.answer-item.incorrect {
  border-left: 4px solid #e74c3c;
}

.question-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.question-number {
  font-weight: bold;
}

.question-status {
  font-weight: bold;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 14px;
}

.question-status.correct {
  background-color: #d5f5e3;
  color: #27ae60;
}

.question-status.incorrect {
  background-color: #fadbd8;
  color: #e74c3c;
}

.question-text {
  font-size: 16px;
  margin-bottom: 10px;
  line-height: 1.5;
}

.answer-details {
  background-color: #f9f9f9;
  padding: 10px;
  border-radius: 4px;
}

.answer-details p {
  margin: 5px 0;
}
</style> 