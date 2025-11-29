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
      searchQuery: '',
      selectedType: '',
      selectedCounterparty: '',
      selectedTags: [],
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
        },
        {
          id: '264919',
          title: 'Акт выполненных работ',
          filename: 'Акт №45/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Ромашка"',
          date: '15.11.2024',
          status: 'Подписан',
          tags: ['Проект Y', 'Финансовый']
        },
        {
          id: '264920',
          title: 'Договор аренды',
          filename: 'Договор №89/2024.pdf',
          type: 'Договор аренды',
          counterparty: 'ООО "Стройсервис"',
          date: '05.08.2024',
          status: 'Активен',
          tags: ['Аренда', 'Юридический', 'Проект Z']
        },
        {
          id: '264921',
          title: 'Счёт на оплату',
          filename: 'Счёт №301.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Ромашка"',
          date: '18.06.2024',
          status: 'Ожидает оплаты',
          tags: ['Финансовый', 'Проект X']
        },
        {
          id: '264922',
          title: 'Договор оказания услуг',
          filename: 'Договор №201/2024.pdf',
          type: 'Договор оказания услуг',
          counterparty: 'ООО "ТехноПрофи"',
          date: '10.04.2024',
          status: 'Активен',
          tags: ['Услуги', 'Технический', 'Проект Y']
        },
        {
          id: '264923',
          title: 'Акт приёма-передачи',
          filename: 'Акт №67/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Вектор"',
          date: '22.05.2024',
          status: 'Подписан',
          tags: ['Передача', 'Юридический', 'Срочный']
        },
        {
          id: '264924',
          title: 'Дополнительное соглашение',
          filename: 'ДС №12/2024.pdf',
          type: 'Дополнительное соглашение',
          counterparty: 'ООО "Ромашка"',
          date: '30.07.2024',
          status: 'Подписано',
          tags: ['Юридический', 'Проект X', 'Изменения']
        },
        {
          id: '264925',
          title: 'Счёт-фактура',
          filename: 'СФ №456.pdf',
          type: 'Счёт-фактура',
          counterparty: 'ООО "Стройсервис"',
          date: '14.03.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'НДС', 'Бухгалтерия']
        },
        {
          id: '264926',
          title: 'Трудовой договор',
          filename: 'ТД №78/2024.pdf',
          type: 'Трудовой договор',
          counterparty: 'ИП Сидоров А.В.',
          date: '20.01.2024',
          status: 'Активен',
          tags: ['Кадровый', 'Сотрудник', 'Юридический']
        },
        {
          id: '264927',
          title: 'Авансовый отчёт',
          filename: 'АО №23/2024.pdf',
          type: 'Авансовый отчёт',
          counterparty: 'ООО "Вектор"',
          date: '08.09.2024',
          status: 'Проведён',
          tags: ['Финансовый', 'Аванс', 'Бухгалтерия']
        },
        {
          id: '264928',
          title: 'Договор подряда',
          filename: 'Договор №302/2024.pdf',
          type: 'Договор подряда',
          counterparty: 'ООО "Строймастер"',
          date: '25.04.2024',
          status: 'Исполняется',
          tags: ['Подряд', 'Строительство', 'Проект Z']
        },
        {
          id: '264929',
          title: 'Счёт на оплату',
          filename: 'Счёт №512.pdf',
          type: 'Счёт',
          counterparty: 'ООО "ТехноПрофи"',
          date: '17.10.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'Технический', 'Срочный']
        },
        {
          id: '264930',
          title: 'Акт сдачи-приёмки',
          filename: 'Акт №89/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Ромашка"',
          date: '03.12.2024',
          status: 'Подписан',
          tags: ['Приёмка', 'Проект X', 'Завершён']
        },
        {
          id: '264931',
          title: 'Лицензионный договор',
          filename: 'Договор №415/2024.pdf',
          type: 'Лицензионный договор',
          counterparty: 'ООО "СофтДев"',
          date: '11.02.2024',
          status: 'Активен',
          tags: ['Лицензия', 'ИТ', 'Юридический']
        },
        {
          id: '264932',
          title: 'Счёт на оплату',
          filename: 'Счёт №623.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Вектор"',
          date: '29.08.2024',
          status: 'Просрочен',
          tags: ['Финансовый', 'Просрочка', 'Срочный']
        },
        {
          id: '264933',
          title: 'Акт выполненных работ',
          filename: 'Акт №156/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Стройсервис"',
          date: '19.07.2024',
          status: 'Подписан',
          tags: ['Строительство', 'Проект Z', 'Финансовый']
        },
        {
          id: '264934',
          title: 'Договор купли-продажи',
          filename: 'Договор №278/2024.pdf',
          type: 'Договор купли-продажи',
          counterparty: 'ООО "Торговая компания"',
          date: '07.05.2024',
          status: 'Исполнен',
          tags: ['Продажа', 'Юридический', 'Завершён']
        },
        {
          id: '264935',
          title: 'Счёт-фактура',
          filename: 'СФ №789.pdf',
          type: 'Счёт-фактура',
          counterparty: 'ООО "Ромашка"',
          date: '21.11.2024',
          status: 'Ожидает оплаты',
          tags: ['Финансовый', 'НДС', 'Проект Y']
        },
        {
          id: '264936',
          title: 'Агентский договор',
          filename: 'Договор №334/2024.pdf',
          type: 'Агентский договор',
          counterparty: 'ООО "Маркетинг Про"',
          date: '13.06.2024',
          status: 'Активен',
          tags: ['Агентство', 'Маркетинг', 'Юридический']
        },
        {
          id: '264937',
          title: 'Акт недостачи',
          filename: 'Акт №44/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Логистик"',
          date: '26.09.2024',
          status: 'На рассмотрении',
          tags: ['Недостача', 'Проблемный', 'Срочный']
        },
        {
          id: '264938',
          title: 'Договор комиссии',
          filename: 'Договор №189/2024.pdf',
          type: 'Договор комиссии',
          counterparty: 'ООО "Трейд"',
          date: '04.03.2024',
          status: 'Активен',
          tags: ['Комиссия', 'Торговля', 'Юридический']
        },
        {
          id: '264939',
          title: 'Счёт на оплату',
          filename: 'Счёт №401.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Стройсервис"',
          date: '16.12.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'Строительство', 'Проект Z']
        },
        {
          id: '264940',
          title: 'Акт сверки',
          filename: 'Акт сверки №5/2024.pdf',
          type: 'Акт сверки',
          counterparty: 'ООО "Вектор"',
          date: '28.02.2024',
          status: 'Подписан',
          tags: ['Сверка', 'Бухгалтерия', 'Финансовый']
        },
        {
          id: '264941',
          title: 'Договор займа',
          filename: 'Договор №76/2024.pdf',
          type: 'Договор займа',
          counterparty: 'ООО "Финансы"',
          date: '09.10.2024',
          status: 'Исполняется',
          tags: ['Займ', 'Финансовый', 'Юридический']
        },
        {
          id: '264942',
          title: 'Счёт на оплату',
          filename: 'Счёт №555.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Ромашка"',
          date: '24.07.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'Проект X', 'Регулярный']
        },
        {
          id: '264943',
          title: 'Акт выполненных работ',
          filename: 'Акт №201/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "ТехноПрофи"',
          date: '31.08.2024',
          status: 'Подписан',
          tags: ['Технический', 'Проект Y', 'Завершён']
        },
        {
          id: '264944',
          title: 'Договор страхования',
          filename: 'Договор №88/2024.pdf',
          type: 'Договор страхования',
          counterparty: 'Страховая компания "Щит"',
          date: '12.01.2024',
          status: 'Активен',
          tags: ['Страхование', 'Юридический', 'Обязательный']
        },
        {
          id: '264945',
          title: 'Счёт-фактура',
          filename: 'СФ №321.pdf',
          type: 'Счёт-фактура',
          counterparty: 'ООО "Строймастер"',
          date: '05.11.2024',
          status: 'Оплачен',
          tags: ['Финансовый', 'НДС', 'Строительство']
        },
        {
          id: '264946',
          title: 'Акт приёма-передачи ОС',
          filename: 'Акт №78/2024.pdf',
          type: 'Акт',
          counterparty: 'ООО "Вектор"',
          date: '19.04.2024',
          status: 'Подписан',
          tags: ['Основные средства', 'Бухгалтерия', 'Юридический']
        }
      ],
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
        // Поиск по тексту
        const searchMatch = !this.searchQuery || 
          document.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          document.filename.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
          document.counterparty.toLowerCase().includes(this.searchQuery.toLowerCase());
        
        // Фильтр по типу
        const typeMatch = !this.selectedType || document.type === this.selectedType;
        
        // Фильтр по контрагенту
        const counterpartyMatch = !this.selectedCounterparty || document.counterparty === this.selectedCounterparty;
        
        // Фильтр по тегам
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
    
    // Методы для загрузки документов
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
        title: fileItem.name.replace(/\.[^/.]+$/, ""), // Убираем расширение файла
        filename: fileItem.name,
        type: this.guessDocumentType(fileItem.name),
        counterparty: 'Новый контрагент',
        date: new Date().toLocaleDateString('ru-RU'),
        status: 'Новый',
        tags: ['Новый']
      };
      
      this.documents.unshift(newDocument);
      this.filterDocuments();
      
      // Автоматически выбираем новый документ
      this.selectedDocument = newDocument;
    },
    
    guessDocumentType(filename) {
      if (filename.includes('договор') || filename.includes('Договор')) return 'Договор';
      if (filename.includes('счет') || filename.includes('Счёт')) return 'Счёт';
      if (filename.includes('акт') || filename.includes('Акт')) return 'Акт';
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
    // Инициализируем отфильтрованные документы
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
</style>

<style scoped src="@/styles/components/MainView.css"></style>