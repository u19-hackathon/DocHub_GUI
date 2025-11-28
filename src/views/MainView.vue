<template>
  <div class="main-view">
    <!-- Боковая панель -->
    <aside class="sidebar">
      <div class="sidebar-header">
        <h1>DocHub</h1>
      </div>
      <nav class="sidebar-nav">
        <a href="#" class="nav-item active">
          <span class="nav-icon">📊</span>
          Dashboard
        </a>
        <a href="#" class="nav-item">
          <span class="nav-icon">📄</span>
          Documents
        </a>
        <a href="#" class="nav-item">
          <span class="nav-icon">📈</span>
          Analytics
        </a>
        <a href="#" class="nav-item">
          <span class="nav-icon">⚙️</span>
          Settings
        </a>
      </nav>
    </aside>

    <!-- Основной контент -->
    <main class="content">
      <!-- Хедер -->
      <header class="content-header">
        <div class="header-actions">
          <div class="search-box">
            <input 
              type="text" 
              placeholder="Search documents..." 
              class="search-input"
            >
          </div>
          <button @click="showUploadModal = true" class="btn btn-primary upload-btn">
              📎 Загрузить документ
            </button>
          <div class="header-buttons">
            <div class="user-menu">
              <span class="user-name">Иван Иванов</span>
              <button @click="handleLogout" class="logout-btn">Выйти</button>
            </div>
          </div>
        </div>
      </header>

      <!-- Фильтры -->
      <div class="filters-section">
        <div class="filters-grid">
          <div class="filter-group">
            <label>Type</label>
            <select class="filter-select">
              <option>Any</option>
              <option>Договор</option>
              <option>Счёт</option>
              <option>Акт</option>
            </select>
          </div>
          <div class="filter-group">
            <label>Tag</label>
            <select class="filter-select">
              <option>Any</option>
              <option>Юридический</option>
              <option>Кадровый</option>
              <option>Финансовый</option>
            </select>
          </div>
          <div class="filter-group">
            <label>Counterparty</label>
            <select class="filter-select">
              <option>All</option>
              <option>ООО "Ромашка"</option>
              <option>ООО "Вектор"</option>
            </select>
          </div>
        </div>
      </div>

      <!-- Основной контент с документами -->
      <div class="main-content">
        <div class="documents-section">
          <div class="section-header">
            <h2>Documents</h2>
          </div>
          
          <!-- Заголовки таблицы -->
          <div class="documents-header">
            <div class="doc-header-column">Document</div>
            <div class="doc-header-column">Type</div>
            <div class="doc-header-column">Counterparty</div>
            <div class="doc-header-column">Date</div>
          </div>

          <!-- Список документов -->
          <div class="documents-list">
            <div 
              v-for="document in documents" 
              :key="document.id"
              class="document-item"
              :class="{ active: selectedDocument?.id === document.id }"
              @click="selectDocument(document)"
            >
              <div class="doc-column document-name">
                <div class="doc-icon">📄</div>
                <div class="doc-info">
                  <div class="doc-title">{{ document.title }}</div>
                  <div class="doc-filename">{{ document.filename }}</div>
                </div>
              </div>
              <div class="doc-column doc-type">{{ document.type }}</div>
              <div class="doc-column doc-counterparty">{{ document.counterparty }}</div>
              <div class="doc-column doc-date">{{ document.date }}</div>
            </div>
          </div>
        </div>

        <!-- Панель предпросмотра документа -->
        <div class="preview-section" v-if="selectedDocument">
          <div class="preview-header">
            <h3>PDF</h3>
            <div class="document-title">{{ selectedDocument.filename }}</div>
          </div>
          
          <div class="document-details">
            <div class="detail-item">
              <label>ID</label>
              <span>{{ selectedDocument.id }}</span>
            </div>
            <div class="detail-item">
              <label>Type</label>
              <span>{{ selectedDocument.type }}</span>
            </div>
            <div class="detail-item">
              <label>Counterparty</label>
              <span>{{ selectedDocument.counterparty }}</span>
            </div>
            <div class="detail-item">
              <label>Date</label>
              <span>{{ selectedDocument.date }}</span>
            </div>
            <div class="detail-item">
              <label>Status</label>
              <span class="status-badge">{{ selectedDocument.status }}</span>
            </div>
            <div class="detail-item tags">
              <label>Tags</label>
              <div class="tags-list">
                <span 
                  v-for="tag in selectedDocument.tags" 
                  :key="tag"
                  class="tag"
                >
                  {{ tag }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Модальное окно загрузки документов -->
    <div v-if="showUploadModal" class="upload-modal-overlay" @click="showUploadModal = false">
      <div class="upload-modal" @click.stop>
        <div class="upload-modal-header">
          <h2>Загрузка документов</h2>
          <button class="close-btn" @click="showUploadModal = false">×</button>
        </div>
        
        <div class="upload-area" 
             @dragover.prevent="dragOver = true"
             @dragleave="dragOver = false"
             @drop="handleFileDrop"
             :class="{ 'drag-over': dragOver }">
          <div class="upload-icon">📤</div>
          <h3>Перетащите файлы сюда</h3>
          <p>или</p>
          <input 
            type="file" 
            ref="fileInput"
            @change="handleFileSelect"
            multiple 
            class="file-input"
            accept=".pdf,.doc,.docx,.xls,.xlsx,.jpg,.png"
          >
          <button class="btn btn-outline" @click="triggerFileInput">
            Выбрать файл
          </button>
        </div>

        <!-- Список загружаемых файлов -->
        <div class="upload-list" v-if="uploadQueue.length > 0">
          <div class="upload-list-header">
            <span>Идёт загрузка {{ uploadQueue.filter(f => f.status !== 'completed').length }} из {{ uploadQueue.length }}</span>
          </div>
          
          <div class="upload-items">
            <div v-for="file in uploadQueue" :key="file.id" class="upload-item">
              <div class="file-info">
                <div class="file-icon">📄</div>
                <div class="file-details">
                  <div class="file-name">{{ file.name }}</div>
                  <div class="file-status">
                    <span v-if="file.status === 'uploading'">Классификация...</span>
                    <span v-else-if="file.status === 'processing'">Сканирование текста... {{ file.progress }}%</span>
                    <span v-else-if="file.status === 'completed'" class="status-completed">Готово</span>
                    <span v-else-if="file.status === 'waiting'" class="status-waiting">Ожидание</span>
                  </div>
                </div>
              </div>
              <div class="file-actions">
                <button v-if="file.status === 'waiting'" @click="removeFromQueue(file.id)" class="btn-remove">×</button>
                <div v-else class="file-progress">
                  <div v-if="file.status === 'uploading' || file.status === 'processing'" class="progress-bar">
                    <div class="progress-fill" :style="{ width: file.progress + '%' }"></div>
                  </div>
                </div>
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
  name: 'MainView',
  data() {
    return {
      selectedDocument: null,
      showUploadModal: false,
      dragOver: false,
      uploadQueue: [],
      documents: [
        {
          id: '264917',
          title: 'Договор поставки',
          filename: 'Договор №154/2024.pdf',
          type: 'Договор поставки',
          counterparty: 'ООО "Ромашка"',
          date: '12.02.2024',
          status: 'На оплате',
          tags: ['Проект X', 'Юридический', 'Поставка']
        },
        {
          id: '264918',
          title: 'Счёт на оплату',
          filename: 'Счёт №287.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Вектор"',
          date: '23.03.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'Срочный']
        }
      ]
    }
  },
  methods: {
    handleLogout() {
      this.$router.push('/login')
    },
    selectDocument(document) {
      this.selectedDocument = document
    },
    
    // Методы для загрузки документов
    triggerFileInput() {
      this.$refs.fileInput.click()
    },
    
    handleFileSelect(event) {
      const files = Array.from(event.target.files)
      this.addFilesToQueue(files)
      event.target.value = '' // Сбрасываем input
    },
    
    handleFileDrop(event) {
      this.dragOver = false
      const files = Array.from(event.dataTransfer.files)
      this.addFilesToQueue(files)
    },
    
    addFilesToQueue(files) {
      files.forEach(file => {
        const fileItem = {
          id: Date.now() + Math.random(),
          name: file.name,
          file: file,
          status: 'waiting',
          progress: 0
        }
        this.uploadQueue.push(fileItem)
      })
      
      // Автоматически начинаем загрузку
      this.processUploadQueue()
    },
    
    async processUploadQueue() {
      const waitingFiles = this.uploadQueue.filter(f => f.status === 'waiting')
      
      for (const fileItem of waitingFiles) {
        fileItem.status = 'uploading'
        
        // Имитация загрузки
        await this.simulateUpload(fileItem)
        
        // После загрузки - классификация
        fileItem.status = 'processing'
        await this.simulateProcessing(fileItem)
        
        // Завершено
        fileItem.status = 'completed'
        fileItem.progress = 100
      }
    },
    
    simulateUpload(fileItem) {
      return new Promise((resolve) => {
        let progress = 0
        const interval = setInterval(() => {
          progress += 10
          fileItem.progress = progress
          
          if (progress >= 100) {
            clearInterval(interval)
            resolve()
          }
        }, 200)
      })
    },
    
    simulateProcessing(fileItem) {
      return new Promise((resolve) => {
        let progress = 0
        const interval = setInterval(() => {
          progress += 15
          fileItem.progress = progress
          
          if (progress >= 100) {
            clearInterval(interval)
            resolve()
          }
        }, 300)
      })
    },
    
    removeFromQueue(fileId) {
      this.uploadQueue = this.uploadQueue.filter(f => f.id !== fileId)
    }
  },
  mounted() {
    if (this.documents.length > 0) {
      this.selectedDocument = this.documents[0]
    }
  }
}
</script>

<style scoped src="@/styles/components/MainView.css"></style>