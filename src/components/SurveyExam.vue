<template>
  <div class="survey-container">
    <!-- 页面1：欢迎页面 -->
    <div v-if="currentPage === 'welcome'" class="welcome-page">
      <h2>欢迎参加宠无忧主播岗位初试</h2>
      
      <div class="company-intro">
        <h3>企业介绍</h3>
        <p>宠无忧（辽宁省）科技有限公司成立于2024年，2025年在自贸区"东北亚宠物交易中心"项下打造"宠无忧云仓小镇"产业综合体，形成覆盖宠物活体QIC服务中心、物流配套、检验检疫及证件签发、暂养及疫苗接种、宠物活体交易合规、电商售后服务支撑、目的地城市线下售后保障、售后履约先行赔付、针对大宗合作伙伴的账期支持、流浪动物救助计划、电商孵化、新零售交易纠纷取证及相关法律服务等全产业链生态体系。产业园区总规划面积10万平方米，包含QIC检验服务中心、交易中心、直播基地、智慧云仓、宠物医疗中心等功能区；产品仓储区域及相关配套设施。</p>
      </div>
      
      <div class="exam-intro">
        <h3>考试介绍</h3>
        <p>本次考试为宠无忧主播岗位初试，您将回答18道问题，每道题5分，共90分，75分以上视为通过。</p>
        
        <div class="exam-notes">
          <h4>注意事项：</h4>
          <ul>
            <li>考试时间为10分钟，请在规定时间内答完试卷并提交，超过时间后无法提交。</li>
            <li>考试过程中不能离开当前浏览器页面，否则视为成绩无效。</li>
            <li>同一个手机号只能参加1次考试，多次考试无法提交。</li>
          </ul>
        </div>
      </div>
      
      <button @click="goToUserInfo" class="btn btn-primary btn-lg">开始考试</button>
    </div>
    
    <!-- 页面2：个人信息填写页 -->
    <div v-else-if="currentPage === 'userInfo'" class="user-info-page">
      <h2>请填写个人信息</h2>
      
      <div class="form-container">
        <div class="form-group">
          <label for="name">姓名</label>
          <input 
            type="text" 
            id="name" 
            v-model="userInfo.student_name" 
            placeholder="请输入您的姓名"
            :class="{ 'invalid-input': submitted && !userInfo.student_name }"
          >
          <span class="error-message" v-if="submitted && !userInfo.student_name">请输入姓名</span>
        </div>
        
        <div class="form-group">
          <label for="phone">手机</label>
          <input 
            type="tel" 
            id="phone" 
            v-model="userInfo.phone" 
            placeholder="请输入您的手机号码"
            :class="{ 'invalid-input': submitted && !isValidPhone }"
          >
          <span class="error-message" v-if="submitted && !isValidPhone">请输入有效的手机号码</span>
        </div>
        
        <div class="form-group">
          <label for="organization">机构</label>
          <input 
            type="text" 
            id="organization" 
            v-model="userInfo.organization" 
            placeholder="请输入您所在的机构"
            :class="{ 'invalid-input': submitted && !userInfo.organization }"
          >
          <span class="error-message" v-if="submitted && !userInfo.organization">请输入机构名称</span>
        </div>
        
        <div class="form-group">
          <label for="remarks">备注</label>
          <textarea 
            id="remarks" 
            v-model="userInfo.remarks" 
            placeholder="其他说明（选填）"
          ></textarea>
        </div>
      </div>
      
      <div class="actions">
        <button @click="goToWelcome" class="btn btn-secondary">返回</button>
        <button @click="validateAndStartExam" class="btn btn-primary">进入考试</button>
      </div>
    </div>
    
    <!-- 页面3：试卷页面 -->
    <div v-else-if="currentPage === 'exam'" class="exam-page">
      <div v-if="loading" class="loading">
        <p>加载试题中...</p>
      </div>
      <div v-else>
        <div class="exam-header">
          <div class="timer-container">
            <div class="timer" :class="{ 'timer-warning': remainingTime <= 120 }">
              剩余时间: {{ formatTime(remainingTime) }}
            </div>
          </div>
          <div class="progress-container">
            <div class="progress-bar">
              <div class="progress" :style="{ width: progress + '%' }"></div>
            </div>
            <div class="progress-text">{{ currentQuestionIndex + 1 }} / {{ questions.length }}</div>
          </div>
        </div>

        <div class="question-container">
          <h3>{{ currentQuestionIndex + 1 }}. {{ currentQuestion.question_text }}</h3>
          
          <div class="options">
            <div 
              class="option"
              :class="{ selected: userAnswers[currentQuestion.id] === 'A' }"
              @click="selectOption(currentQuestion.id, 'A')"
            >
              <div class="radio-btn">
                <div class="inner-circle" v-if="userAnswers[currentQuestion.id] === 'A'"></div>
              </div>
              <span>{{ currentQuestion.option_a }}</span>
            </div>
            
            <div 
              class="option"
              :class="{ selected: userAnswers[currentQuestion.id] === 'B' }"
              @click="selectOption(currentQuestion.id, 'B')"
            >
              <div class="radio-btn">
                <div class="inner-circle" v-if="userAnswers[currentQuestion.id] === 'B'"></div>
              </div>
              <span>{{ currentQuestion.option_b }}</span>
            </div>
            
            <div 
              class="option"
              :class="{ selected: userAnswers[currentQuestion.id] === 'C' }"
              @click="selectOption(currentQuestion.id, 'C')"
            >
              <div class="radio-btn">
                <div class="inner-circle" v-if="userAnswers[currentQuestion.id] === 'C'"></div>
              </div>
              <span>{{ currentQuestion.option_c }}</span>
            </div>
            
            <div 
              class="option"
              :class="{ selected: userAnswers[currentQuestion.id] === 'D' }"
              @click="selectOption(currentQuestion.id, 'D')"
            >
              <div class="radio-btn">
                <div class="inner-circle" v-if="userAnswers[currentQuestion.id] === 'D'"></div>
              </div>
              <span>{{ currentQuestion.option_d }}</span>
            </div>
          </div>
        </div>

        <div class="actions">
          <button 
            v-if="currentQuestionIndex > 0" 
            @click="prevQuestion" 
            class="btn btn-secondary"
          >
            上一题
          </button>
          <button 
            v-if="currentQuestionIndex < questions.length - 1" 
            @click="nextQuestion" 
            class="btn btn-primary"
          >
            下一题
          </button>
          <button 
            v-else 
            @click="submitExam" 
            class="btn btn-submit"
          >
            提交试卷
          </button>
        </div>
        
        <div class="question-navigation">
          <div 
            v-for="(question, index) in questions" 
            :key="index"
            class="question-number"
            :class="{ 
              'current': currentQuestionIndex === index,
              'answered': userAnswers[question.id] !== undefined
            }"
            @click="goToQuestion(index)"
          >
            {{ index + 1 }}
          </div>
        </div>
      </div>
    </div>
    
    <!-- 页面4：考试结果页 -->
    <div v-else-if="currentPage === 'result'" class="result-page">
      <div v-if="examResult.percentage >= 75" class="result-pass">
        <h2>恭喜您通过考试！</h2>
        <div class="score-display">{{ examResult.score }} / {{ examResult.total_questions * 5 }}</div>
        <p>请等待工作人员电话通知面试时间。</p>
      </div>
      
      <div v-else class="result-fail">
        <h2>很遗憾，您未通过考试</h2>
        <div class="score-display">{{ examResult.score }} / {{ examResult.total_questions * 5 }}</div>
        <p>通过分数线：75分</p>
      </div>
      
      <div class="user-info-display">
        <h3>考生信息</h3>
        <p><strong>姓名：</strong>{{ examResult.student_name }}</p>
        <p><strong>手机：</strong>{{ examResult.phone }}</p>
        <p v-if="examResult.organization"><strong>机构：</strong>{{ examResult.organization }}</p>
      </div>
      
      <button @click="acknowledgeResult" class="btn btn-primary">好的</button>
    </div>
    
    <!-- 错误提示模态窗口 -->
    <div v-if="showErrorModal" class="error-modal-overlay">
      <div class="error-modal">
        <h3>提示</h3>
        <p>{{ errorMessage }}</p>
        <button @click="closeErrorModal" class="btn btn-primary">确定</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SurveyExam',
  data() {
    return {
      currentPage: 'welcome',
      currentQuestionIndex: 0,
      submitted: false,
      examTime: 600, // 10分钟 = 600秒
      remainingTime: 600,
      timer: null,
      pageLeaveDetected: false,
      loading: false,
      apiBaseUrl: '/api',
      showErrorModal: false,
      errorMessage: '',
      userInfo: {
        student_name: '',
        student_id: 'DEFAULT_ID',
        phone: '',
        organization: '',
        remarks: ''
      },
      questions: [],
      userAnswers: {},
      examResult: {
        student_name: '',
        student_id: '',
        phone: '',
        organization: '',
        remarks: '',
        score: 0,
        total_questions: 0,
        percentage: 0
      }
    }
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex] || {};
    },
    isValidPhone() {
      const phoneRegex = /^1[3-9]\d{9}$/;
      return phoneRegex.test(this.userInfo.phone);
    },
    progress() {
      return this.questions.length ? (this.currentQuestionIndex / (this.questions.length - 1)) * 100 : 0;
    }
  },
  methods: {
    goToWelcome() {
      this.currentPage = 'welcome';
    },
    goToUserInfo() {
      this.currentPage = 'userInfo';
    },
    validateAndStartExam() {
      this.submitted = true;
      
      if (this.userInfo.student_name && this.isValidPhone && this.userInfo.organization) {
        this.loadQuestions();
      }
    },
    async loadQuestions() {
      this.loading = true;
      try {
        const response = await fetch(`${this.apiBaseUrl}/questions`);
        if (!response.ok) {
          throw new Error('获取试题失败');
        }
        const data = await response.json();
        if (data.success && data.questions && data.questions.length > 0) {
          this.questions = data.questions;
          this.currentPage = 'exam';
          this.startTimer();
          this.setupPageLeaveDetection();
        } else {
          this.showError('获取试题失败，请稍后再试');
        }
      } catch (error) {
        console.error('加载试题出错:', error);
        this.showError(`获取试题失败: ${error.message || '请检查网络连接'}`);
      } finally {
        this.loading = false;
      }
    },
    startTimer() {
      this.remainingTime = this.examTime;
      clearInterval(this.timer);
      
      this.timer = setInterval(() => {
        this.remainingTime--;
        
        if (this.remainingTime <= 0) {
          clearInterval(this.timer);
          this.submitExam();
        }
      }, 1000);
    },
    setupPageLeaveDetection() {
      window.addEventListener('beforeunload', this.handlePageLeave);
      window.addEventListener('visibilitychange', this.handleVisibilityChange);
    },
    removePageLeaveDetection() {
      window.removeEventListener('beforeunload', this.handlePageLeave);
      window.removeEventListener('visibilitychange', this.handleVisibilityChange);
    },
    handlePageLeave(event) {
      if (this.currentPage === 'exam') {
        this.pageLeaveDetected = true;
        event.preventDefault();
        event.returnValue = '离开页面将视为考试作废，确定要离开吗？';
        return event.returnValue;
      }
    },
    handleVisibilityChange() {
      if (document.visibilityState === 'hidden' && this.currentPage === 'exam') {
        this.pageLeaveDetected = true;
      }
    },
    formatTime(seconds) {
      const minutes = Math.floor(seconds / 60);
      const remainingSeconds = seconds % 60;
      return `${minutes}:${remainingSeconds < 10 ? '0' : ''}${remainingSeconds}`;
    },
    selectOption(questionId, option) {
      this.$set(this.userAnswers, questionId, option);
    },
    nextQuestion() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++;
      }
    },
    prevQuestion() {
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--;
      }
    },
    goToQuestion(index) {
      this.currentQuestionIndex = index;
    },
    showError(message) {
      this.errorMessage = message;
      this.showErrorModal = true;
    },
    closeErrorModal() {
      this.showErrorModal = false;
      this.errorMessage = '';
    },
    async submitExam() {
      clearInterval(this.timer);
      this.removePageLeaveDetection();
      
      // 构建提交数据
      const submitData = {
        ...this.userInfo,
        answers: this.userAnswers
      };
      
      try {
        const response = await fetch(`${this.apiBaseUrl}/submit`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(submitData),
        });
        
        const result = await response.json();
        
        if (result.success) {
          this.examResult = result;
          this.currentPage = 'result';
        } else {
          // 处理有明确错误信息的失败情况
          const errorMessage = result.message || '未知错误';
          if (errorMessage.includes('手机号') || errorMessage.includes('已参加')) {
            this.showError('该手机号已参加过考试，每个手机号只能参加一次考试');
          } else {
            this.showError(`提交考试答案失败: ${errorMessage}`);
          }
          // 如果提交失败，回到考试页面
          this.startTimer();
          this.setupPageLeaveDetection();
        }
      } catch (error) {
        console.error('提交考试答案出错:', error);
        
        // 尝试从响应中提取更多信息
        if (error.response) {
          try {
            const errorData = await error.response.json();
            if (errorData && errorData.message) {
              if (errorData.message.includes('手机号') || errorData.message.includes('已参加')) {
                this.showError('该手机号已参加过考试，每个手机号只能参加一次考试');
              } else {
                this.showError(`提交考试答案失败: ${errorData.message}`);
              }
            } else {
              this.showError(`提交考试答案失败: ${error.message || '服务器错误'}`);
            }
          } catch (e) {
            this.showError(`提交考试答案失败: ${error.message || '服务器错误'}`);
          }
        } else {
          this.showError(`提交考试答案失败: ${error.message || '网络错误，请检查网络连接'}`);
        }
        
        // 如果提交失败，回到考试页面
        this.startTimer();
        this.setupPageLeaveDetection();
      }
    },
    resetExam() {
      this.currentPage = 'welcome';
      this.currentQuestionIndex = 0;
      this.submitted = false;
      this.pageLeaveDetected = false;
      this.questions = [];
      this.userAnswers = {};
      
      // 重置用户信息
      this.userInfo = {
        student_name: '',
        student_id: 'DEFAULT_ID',
        phone: '',
        organization: '',
        remarks: ''
      };
      
      // 重置考试结果
      this.examResult = {
        student_name: '',
        student_id: '',
        phone: '',
        organization: '',
        remarks: '',
        score: 0,
        total_questions: 0,
        percentage: 0
      };
    },
    acknowledgeResult() {
      this.showError('感谢您参加本次考试！每个手机号只能参加一次考试。');
    }
  },
  beforeDestroy() {
    clearInterval(this.timer);
    this.removePageLeaveDetection();
  }
}
</script>

<style scoped>
.survey-container {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  padding: 30px;
  margin-bottom: 30px;
}

/* 欢迎页面样式 */
.welcome-page {
  text-align: center;
}

.company-intro, .exam-intro {
  margin-bottom: 30px;
  text-align: left;
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
}

.company-intro h3, .exam-intro h3 {
  color: #42b983;
  margin-bottom: 15px;
}

.company-intro p, .exam-intro p {
  line-height: 1.6;
  color: #495057;
}

.exam-notes {
  background-color: #fff8f0;
  border-left: 4px solid #f0ad4e;
  padding: 15px;
  margin-top: 15px;
  border-radius: 4px;
}

.exam-notes h4 {
  color: #f0ad4e;
  margin-bottom: 10px;
  font-weight: 600;
}

.exam-notes ul {
  padding-left: 20px;
  margin: 0;
}

.exam-notes li {
  margin-bottom: 8px;
  color: #555;
  line-height: 1.5;
}

.btn-lg {
  padding: 12px 30px;
  font-size: 18px;
  margin-top: 20px;
}

/* 个人信息页面样式 */
.user-info-page h2 {
  text-align: center;
  margin-bottom: 30px;
  color: #42b983;
}

.form-container {
  max-width: 600px;
  margin: 0 auto 30px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #495057;
}

.form-group input, .form-group textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
}

.form-group textarea {
  height: 100px;
  resize: vertical;
}

.invalid-input {
  border-color: #dc3545 !important;
}

.error-message {
  color: #dc3545;
  font-size: 14px;
  margin-top: 5px;
  display: block;
}

/* 考试页面样式 */
.loading {
  text-align: center;
  padding: 40px;
  font-size: 18px;
  color: #495057;
}

.exam-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.timer-container {
  flex: 1;
}

.timer {
  font-size: 18px;
  font-weight: bold;
  color: #495057;
  background-color: #f8f9fa;
  padding: 8px 15px;
  border-radius: 4px;
  display: inline-block;
}

.timer-warning {
  color: #dc3545;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% { opacity: 1; }
  50% { opacity: 0.7; }
  100% { opacity: 1; }
}

.progress-container {
  flex: 2;
  margin-left: 20px;
}

.progress-bar {
  height: 8px;
  background-color: #e9ecef;
  border-radius: 4px;
  overflow: hidden;
}

.progress {
  height: 100%;
  background-color: #42b983;
  transition: width 0.3s ease;
}

.progress-text {
  text-align: right;
  margin-top: 5px;
  font-size: 14px;
  color: #6c757d;
}

.question-container {
  margin-bottom: 30px;
}

.question-container h3 {
  margin-bottom: 20px;
  font-size: 18px;
  color: #333;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.option {
  display: flex;
  align-items: center;
  padding: 12px 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: left;
}

.option:hover {
  border-color: #42b983;
  background-color: #f8f9fa;
}

.option.selected {
  border-color: #42b983;
  background-color: #f0f9f5;
}

.radio-btn {
  width: 20px;
  height: 20px;
  border: 2px solid #42b983;
  border-radius: 50%;
  margin-right: 12px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.inner-circle {
  width: 10px;
  height: 10px;
  background-color: #42b983;
  border-radius: 50%;
}

.question-navigation {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 30px;
  justify-content: center;
}

.question-number {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #ddd;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s ease;
}

.question-number:hover {
  background-color: #f8f9fa;
}

.question-number.current {
  background-color: #42b983;
  color: white;
  border-color: #42b983;
}

.question-number.answered {
  background-color: #f0f9f5;
  border-color: #42b983;
}

.actions {
  display: flex;
  justify-content: space-between;
  margin-top: 30px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: all 0.2s ease;
}

.btn-primary {
  background-color: #42b983;
  color: white;
}

.btn-primary:hover {
  background-color: #3aa876;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #5a6268;
}

.btn-submit {
  background-color: #28a745;
  color: white;
}

.btn-submit:hover {
  background-color: #218838;
}

/* 结果页面样式 */
.result-page {
  text-align: center;
}

.result-pass h2, .result-fail h2 {
  margin-bottom: 20px;
}

.result-pass h2 {
  color: #28a745;
}

.result-fail h2 {
  color: #dc3545;
}

.score-display {
  font-size: 48px;
  font-weight: bold;
  margin: 20px 0;
  color: #495057;
}

.user-info-display {
  margin: 30px 0;
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  text-align: left;
}

.user-info-display h3 {
  color: #42b983;
  margin-bottom: 15px;
}

.user-info-display p {
  margin: 8px 0;
  color: #495057;
}

/* 错误模态窗口样式 */
.error-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.error-modal {
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  max-width: 400px;
  width: 90%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.error-modal h3 {
  color: #dc3545;
  margin-bottom: 15px;
}

.error-modal p {
  margin-bottom: 20px;
  color: #495057;
}

.error-modal button {
  display: block;
  margin: 0 auto;
  min-width: 100px;
}
</style> 