<template>
  <div class="login-view">
    <div class="login-container">
      <div class="login-card">
        <div class="login-header">
          <h1>DocHub</h1>
          <p>Войдите в систему</p>
        </div>
        
        <form @submit.prevent="handleLogin" class="login-form">
          <div class="form-group">
            <label class="form-label">Email</label>
            <input
              v-model="email"
              type="email"
              class="form-input"
              placeholder="Введите ваш email"
              required
            >
          </div>
          
          <div class="form-group">
            <label class="form-label">Пароль</label>
            <input
              v-model="password"
              type="password"
              class="form-input"
              placeholder="Введите пароль"
              required
            >
          </div>
          
          <div v-if="error" class="error-message">
            {{ error }}
          </div>
          
          <button 
            type="submit" 
            class="btn btn-primary login-btn"
            :disabled="loading"
          >
            <span v-if="loading">Вход...</span>
            <span v-else>Войти</span>
          </button>
        </form>

        <div class="register-section">
          <p class="register-text">Нет аккаунта?</p>
          <button 
            @click="goToRegister" 
            class="btn btn-secondary register-btn"
          >
            Зарегистрироваться
          </button>
        </div>
        
        <div class="demo-credentials">
          <p><strong>Демо доступ:</strong></p>
          <p>Email: user@company.com</p>
          <p>Пароль: password</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoginView',
  data() {
    return {
      email: '',
      password: '',
      loading: false,
      error: ''
    }
  },
  methods: {
    async handleLogin() {
      this.loading = true
      this.error = ''
      
      try {
        // Имитация API запроса
        await new Promise((resolve, reject) => {
          setTimeout(() => {
            if (this.email === 'user@company.com' && this.password === 'password') {
              resolve()
            } else {
              reject(new Error('Неверные учетные данные'))
            }
          }, 1000)
        })
        
        // Перенаправление на главную
        this.$router.push('/')
      } catch (error) {
        this.error = error.message
      } finally {
        this.loading = false
      }
    },
    
    goToRegister() {
      this.$router.push('/register')
    }
  }
}
</script>

<style scoped src="@/styles/components/LoginView.css"></style>