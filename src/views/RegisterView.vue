<template>
  <div class="register-view">
    <div class="register-container">
      <div class="register-card">
        <div class="register-header">
          <h1>DocHub</h1>
          <p>Создайте аккаунт</p>
        </div>
        
        <form @submit.prevent="handleRegister" class="register-form">
          <div class="form-group">
            <label class="form-label">ФИО</label>
            <input
              v-model="fullName"
              type="text"
              class="form-input"
              placeholder="Введите ваше ФИО полностью"
              required
            >
          </div>

          <div class="form-group">
            <label class="form-label">Должность</label>
            <input
              v-model="position"
              type="text"
              class="form-input"
              placeholder="Введите вашу должность"
              required
            >
          </div>

          <div class="form-group">
            <label class="form-label">Название организации</label>
            <input
              v-model="organization"
              type="text"
              class="form-input"
              placeholder="Введите название вашей организации"
              required
            >
          </div>
          
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
              placeholder="Придумайте пароль"
              required
            >
          </div>
          
          <div v-if="error" class="error-message">
            {{ error }}
          </div>

          <div v-if="success" class="success-message">
            {{ success }}
          </div>
          
          <button 
            type="submit" 
            class="btn btn-primary register-btn"
            :disabled="loading"
          >
            <span v-if="loading">Регистрация...</span>
            <span v-else>Зарегистрироваться</span>
          </button>
        </form>

        <div class="login-section">
          <p class="login-text">Уже есть аккаунт?</p>
          <button 
            @click="goToLogin" 
            class="btn btn-secondary login-btn"
          >
            Войти
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'RegisterView',
  data() {
    return {
      fullName: '',
      position: '',
      organization: '',
      email: '',
      password: '',
      loading: false,
      error: '',
      success: ''
    }
  },
  methods: {
    async handleRegister() {
      this.loading = true
      this.error = ''
      this.success = ''

      // Валидация
      if (this.password.length < 6) {
        this.error = 'Пароль должен содержать минимум 6 символов'
        this.loading = false
        return
      }

      if (!this.organization.trim()) {
        this.error = 'Название организации обязательно для заполнения'
        this.loading = false
        return
      }
      
      try {
        // Имитация API запроса регистрации
        await new Promise((resolve, reject) => {
          setTimeout(() => {
            // Проверяем, не занят ли email
            if (this.email === 'user@company.com') {
              reject(new Error('Пользователь с таким email уже существует'))
            } else {
              resolve()
            }
          }, 1500)
        })
        
        // Успешная регистрация
        this.success = 'Аккаунт успешно создан! Перенаправляем на страницу входа...'
        
        // Через 2 секунды перенаправляем на страницу входа
        setTimeout(() => {
          this.$router.push('/login')
        }, 2000)
        
      } catch (error) {
        this.error = error.message
      } finally {
        this.loading = false
      }
    },
    
    goToLogin() {
      this.$router.push('/login')
    }
  }
}
</script>

<style scoped src="@/styles/components/RegisterView.css"></style>