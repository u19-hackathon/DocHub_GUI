<template>
  <div class="main-view">
    <!-- Боковая панель -->
    <aside class="sidebar">
      <div class="sidebar-header">
        <h1>DocHub</h1>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" class="nav-item" :class="{ active: $route.name === 'Home' }">
          <span class="nav-icon">📄</span>
          Документы
        </router-link>
        <router-link to="/analytics" class="nav-item" :class="{ active: $route.name === 'Analytics' }">
          <span class="nav-icon">📈</span>
          Аналитика
        </router-link>
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
              v-model="searchQuery"
              placeholder="Поиск документов..." 
              class="search-input"
              @input="filterDocuments"
            >
          </div>
          <button @click="showUploadModal = true" class="btn btn-primary upload-btn">
            📎 Загрузить документ
          </button>
          <div class="user-menu">
            <span class="user-name">Иван Иванов</span>
            <button @click="handleLogout" class="logout-btn">Выйти</button>
          </div>
        </div>
      </header>

      <!-- Фильтры -->
      <div class="filters-section">
        <div class="filters-grid">
          <div class="filter-group">
            <label>Тип</label>
            <select v-model="selectedType" class="filter-select" @change="filterDocuments">
              <option value="">Любой</option>
              <option v-for="type in availableTypes" :key="type" :value="type">{{ type }}</option>
            </select>
          </div>
          <div class="filter-group">
            <label>Контрагент</label>
            <select v-model="selectedCounterparty" class="filter-select" @change="filterDocuments">
              <option value="">Все</option>
              <option v-for="counterparty in availableCounterparties" :key="counterparty" :value="counterparty">
                {{ counterparty }}
              </option>
            </select>
          </div>
        </div>

        <!-- Фильтр по тегам -->
        <div class="tags-filter" v-if="availableTags.length > 0">
          <label>Теги:</label>
          <div class="tags-list-filter">
            <span 
              v-for="tag in availableTags" 
              :key="tag"
              class="tag-filter"
              :class="{ active: selectedTags.includes(tag) }"
              @click="toggleTag(tag)"
            >
              {{ tag }}
            </span>
          </div>
          <button v-if="selectedTags.length > 0" @click="clearTags" class="clear-tags-btn">
            Очистить теги
          </button>
        </div>
      </div>

      <!-- Основной контент с документами -->
      <div class="main-content">
        <div class="documents-section">
          <div class="section-header">
            <h2>Документы ({{ filteredDocuments.length }})</h2>
          </div>
          
          <!-- Заголовки таблицы -->
          <div class="documents-header">
            <div class="doc-header-column">Документ</div>
            <div class="doc-header-column">Тип</div>
            <div class="doc-header-column">Контрагент</div>
            <div class="doc-header-column">Дата</div>
          </div>

          <!-- Список документов -->
          <div class="documents-list">
            <div 
              v-for="document in filteredDocuments" 
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
                  <div class="document-tags-preview">
                    <span 
                      v-for="tag in document.tags" 
                      :key="tag"
                      class="tag-preview"
                    >
                      {{ tag }}
                    </span>
                  </div>
                </div>
              </div>
              <div class="doc-column doc-type">{{ document.type }}</div>
              <div class="doc-column doc-counterparty">{{ document.counterparty }}</div>
              <div class="doc-column doc-date">{{ document.date }}</div>
            </div>
            
            <div v-if="filteredDocuments.length === 0" class="no-documents">
              Документы не найдены
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
              <label>Тип</label>
              <span>{{ selectedDocument.type }}</span>
            </div>
            <div class="detail-item">
              <label>Контрагент</label>
              <span>{{ selectedDocument.counterparty }}</span>
            </div>
            <div class="detail-item">
              <label>Дата</label>
              <span>{{ selectedDocument.date }}</span>
            </div>
            <div class="detail-item">
              <label>Статус</label>
              <span class="status-badge">{{ selectedDocument.status }}</span>
            </div>
            <div class="detail-item tags">
              <label>Теги</label>
              <div class="tags-list">
                <span 
                  v-for="tag in selectedDocument.tags" 
                  :key="tag"
                  class="tag"
                  :class="{ active: selectedTags.includes(tag) }"
                  @click="toggleTag(tag)"
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

        <!-- Настройки загрузки -->
        <div class="upload-settings" v-if="uploadQueue.length > 0">
          <div class="settings-section">
            <h4>Настройки документов</h4>
            
            <div class="setting-group">
              <label>Контрагент</label>
              <input 
                type="text" 
                v-model="uploadCounterparty"
                placeholder="Введите название контрагента"
                class="setting-input"
              >
            </div>

            <div class="setting-group">
              <label>Статус</label>
              <select v-model="uploadStatus" class="setting-select">
                <option value="">Выберите статус</option>
                <option v-for="status in availableStatuses" :key="status" :value="status">
                  {{ status }}
                </option>
              </select>
              <input 
                v-if="uploadStatus === 'custom'"
                type="text" 
                v-model="customStatus"
                placeholder="Введите свой статус"
                class="setting-input"
                style="margin-top: 8px;"
              >
            </div>
            
            <div class="setting-group">
              <label>Теги</label>
              <div class="tags-input-container">
                <div class="selected-tags">
                  <span 
                    v-for="tag in uploadTags" 
                    :key="tag"
                    class="upload-tag"
                  >
                    {{ tag }}
                    <span @click="removeUploadTag(tag)" class="remove-tag">×</span>
                  </span>
                </div>
                <input 
                  type="text" 
                  v-model="newTag"
                  @keydown.enter="addUploadTag"
                  @keydown.backspace="handleBackspace"
                  placeholder="Введите тег и нажмите Enter"
                  class="tag-input"
                >
              </div>
              <div class="tags-suggestions" v-if="tagSuggestions.length > 0 && newTag">
                <div 
                  v-for="suggestion in tagSuggestions"
                  :key="suggestion"
                  class="tag-suggestion"
                  @click="selectTagSuggestion(suggestion)"
                >
                  {{ suggestion }}
                </div>
              </div>
            </div>
          </div>
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

          <div class="upload-actions">
            <button @click="startUpload" class="btn btn-primary" :disabled="uploadQueue.some(f => f.status !== 'waiting')">
              Начать загрузку
            </button>
            <button @click="clearUploadQueue" class="btn btn-outline">
              Очистить очередь
            </button>
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
      searchQuery: '',
      selectedType: '',
      selectedCounterparty: '',
      selectedTags: [],
      // Настройки загрузки
      uploadCounterparty: '',
      uploadStatus: 'Новый',
      customStatus: '',
      uploadTags: [],
      newTag: '',
      documents: [],
      filteredDocuments: []
    }
  },
  computed: {
    availableTypes() {
      return [...new Set(this.documents.map(doc => doc.type))];
    },
    availableCounterparties() {
      return [...new Set(this.documents.map(doc => doc.counterparty))];
    },
    availableTags() {
      const allTags = this.documents.flatMap(doc => doc.tags);
      return [...new Set(allTags)].sort();
    },
    availableStatuses() {
      const statuses = [...new Set(this.documents.map(doc => doc.status))];
      return [...statuses, 'custom'].sort();
    },
    tagSuggestions() {
      if (!this.newTag) return [];
      return this.availableTags.filter(tag => 
        tag.toLowerCase().includes(this.newTag.toLowerCase()) && 
        !this.uploadTags.includes(tag)
      ).slice(0, 5);
    },
    finalUploadStatus() {
      if (this.uploadStatus === 'custom') {
        return this.customStatus || 'Новый';
      }
      return this.uploadStatus || 'Новый';
    }
  },
  methods: {
    handleLogout() {
      this.$router.push('/login')
    },
    selectDocument(document) {
      this.selectedDocument = document
    },
    
    // Методы для фильтрации
    filterDocuments() {
      this.filteredDocuments = this.documents.filter(document => {
        const searchMatch = !this.searchQuery || 
          document.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          document.filename.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          document.counterparty.toLowerCase().includes(this.searchQuery.toLowerCase());
        
        const typeMatch = !this.selectedType || document.type === this.selectedType;
        const counterpartyMatch = !this.selectedCounterparty || document.counterparty === this.selectedCounterparty;
        const tagsMatch = this.selectedTags.length === 0 || 
          this.selectedTags.every(tag => document.tags.includes(tag));
        
        return searchMatch && typeMatch && counterpartyMatch && tagsMatch;
      });
    },
    
    toggleTag(tag) {
      const index = this.selectedTags.indexOf(tag);
      if (index > -1) {
        this.selectedTags.splice(index, 1);
      } else {
        this.selectedTags.push(tag);
      }
      this.filterDocuments();
    },
    
    clearTags() {
      this.selectedTags = [];
      this.filterDocuments();
    },
    
    // Методы для загрузки документов с тегами
    triggerFileInput() {
      this.$refs.fileInput.click()
    },
    
    handleFileSelect(event) {
      const files = Array.from(event.target.files)
      this.addFilesToQueue(files)
      event.target.value = ''
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
    },
    
    startUpload() {
      this.processUploadQueue()
    },
    
    async processUploadQueue() {
      const waitingFiles = this.uploadQueue.filter(f => f.status === 'waiting')
      
      for (const fileItem of waitingFiles) {
        fileItem.status = 'uploading'
        await this.simulateUpload(fileItem)
        fileItem.status = 'processing'
        await this.simulateProcessing(fileItem)
        fileItem.status = 'completed'
        fileItem.progress = 100
        
        // Добавляем новый документ в список после загрузки
        this.addNewDocument(fileItem)
      }
    },
    
    addNewDocument(fileItem) {
      const newDocument = {
        id: Date.now().toString(),
        title: fileItem.name.replace(/\.[^/.]+$/, ""),
        filename: fileItem.name,
        type: this.guessDocumentType(fileItem.name),
        counterparty: this.uploadCounterparty || 'Не указан',
        date: new Date().toLocaleDateString('ru-RU'),
        status: this.finalUploadStatus,
        tags: [...this.uploadTags]
      };
      
      this.documents.unshift(newDocument);
      this.filterDocuments();
      this.selectedDocument = newDocument;
    },

    clearUploadQueue() {
      this.uploadQueue = [];
      this.uploadTags = [];
      this.uploadCounterparty = '';
      this.uploadStatus = 'Новый';
      this.customStatus = '';
      this.newTag = '';
    },
    
    // Методы для работы с тегами при загрузке
    addUploadTag() {
      if (this.newTag.trim() && !this.uploadTags.includes(this.newTag.trim())) {
        this.uploadTags.push(this.newTag.trim());
        this.newTag = '';
      }
    },
    
    removeUploadTag(tag) {
      this.uploadTags = this.uploadTags.filter(t => t !== tag);
    },
    
    selectTagSuggestion(suggestion) {
      if (!this.uploadTags.includes(suggestion)) {
        this.uploadTags.push(suggestion);
      }
      this.newTag = '';
    },
    
    handleBackspace(event) {
      if (event.target.value === '' && this.uploadTags.length > 0) {
        this.uploadTags.pop();
      }
    },
    
    guessDocumentType(filename) {
      if (filename.includes('договор') || filename.includes('Договор')) return 'Договор';
      if (filename.includes('счет') || filename.includes('Счёт')) return 'Счёт';
      if (filename.includes('акт') || filename.includes('Акт')) return 'Акт';
      if (filename.includes('отчёт') || filename.includes('Отчёт')) return 'Отчёт';
      if (filename.includes('фактура') || filename.includes('Фактура')) return 'Счёт-фактура';
      return 'Другой';
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
    this.filteredDocuments = [...this.documents];
    if (this.documents.length > 0) {
      this.selectedDocument = this.documents[0]
    }
  }
}
</script>

<style scoped>
/* Стили для фильтра по тегам */
.tags-filter {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #e1e5e9;
}

.tags-filter label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #2c3e50;
  font-size: 14px;
}

.tags-list-filter {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 10px;
}

.tag-filter {
  background: #e3f2fd;
  color: #1976d2;
  padding: 6px 12px;
  border-radius: 16px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s;
  border: 2px solid transparent;
}

.tag-filter:hover {
  background: #bbdefb;
}

.tag-filter.active {
  background: #1976d2;
  color: white;
  border-color: #1565c0;
}

.clear-tags-btn {
  background: none;
  border: 1px solid #dc3545;
  color: #dc3545;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.3s;
}

.clear-tags-btn:hover {
  background: #dc3545;
  color: white;
}

/* Стили для тегов в списке документов */
.document-tags-preview {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  margin-top: 4px;
}

.tag-preview {
  background: #f8f9fa;
  color: #6c757d;
  padding: 2px 6px;
  border-radius: 10px;
  font-size: 10px;
  border: 1px solid #e1e5e9;
}

/* Стили для сообщения об отсутствии документов */
.no-documents {
  text-align: center;
  padding: 40px;
  color: #6c757d;
  font-style: italic;
}

/* Стили для кликабельных тегов в превью */
.tags-list .tag {
  cursor: pointer;
  transition: all 0.3s;
}

.tags-list .tag:hover {
  background: #bbdefb;
}

.tags-list .tag.active {
  background: #1976d2;
  color: white;
}

/* Стили для настроек загрузки */
.upload-settings {
  padding: 0 24px 24px;
}

.settings-section {
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e1e5e9;
}

.settings-section h4 {
  margin: 0 0 16px 0;
  color: #2c3e50;
  font-size: 16px;
  font-weight: 600;
}

.setting-group {
  margin-bottom: 16px;
}

.setting-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #2c3e50;
  font-size: 14px;
}

.setting-input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.setting-input:focus {
  outline: none;
  border-color: #3498db;
}

/* Стили для ввода тегов */
.tags-input-container {
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 8px;
  background: white;
  min-height: 44px;
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 6px;
}

.selected-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.upload-tag {
  background: #e3f2fd;
  color: #1976d2;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
}

.remove-tag {
  cursor: pointer;
  font-weight: bold;
  font-size: 14px;
  line-height: 1;
  padding-left: 4px;
}

.remove-tag:hover {
  color: #dc3545;
}

.tag-input {
  border: none;
  outline: none;
  padding: 6px;
  font-size: 14px;
  flex: 1;
  min-width: 120px;
}

.tags-suggestions {
  position: absolute;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  margin-top: 4px;
  max-height: 150px;
  overflow-y: auto;
  z-index: 10;
  width: 100%;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.tag-suggestion {
  padding: 8px 12px;
  cursor: pointer;
  font-size: 14px;
  border-bottom: 1px solid #f0f0f0;
}

.tag-suggestion:hover {
  background: #f8f9fa;
}

.tag-suggestion:last-child {
  border-bottom: none;
}

/* Стили для кнопок загрузки */
.upload-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  padding-top: 16px;
  border-top: 1px solid #e1e5e9;
  margin-top: 16px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s;
}

.btn-primary {
  background: #3498db;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #2980b9;
}

.btn-primary:disabled {
  background: #bdc3c7;
  cursor: not-allowed;
}

.btn-outline {
  background: white;
  color: #3498db;
  border: 1px solid #3498db;
}

.btn-outline:hover {
  background: #3498db;
  color: white;
}
</style>

<style scoped src="@/styles/components/MainView.css"></style>