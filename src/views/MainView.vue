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
  </div>
</template>

<script>
export default {
  name: 'MainView',
  data() {
    return {
      selectedDocument: null,
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
          filename: 'Акт №45.pdf',
          type: 'Акт',
          counterparty: 'ООО "Ромашка"',
          date: '19.02.2024',
          status: 'Подписан',
          tags: ['Проект X', 'Отчётность']
        },
        {
          id: '264920',
          title: 'Договор поставки',
          filename: 'Договор №155.pdf',
          type: 'Договор поставки',
          counterparty: 'HR Department',
          date: '15.03.2024',
          status: 'Черновик',
          tags: ['Кадровый', 'Внутренний']
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
    }
  },
  mounted() {
    // Автоматически выбираем первый документ при загрузке
    if (this.documents.length > 0) {
      this.selectedDocument = this.documents[0]
    }
  }
}
</script>

<style scoped src="@/styles/components/MainView.css"></style>