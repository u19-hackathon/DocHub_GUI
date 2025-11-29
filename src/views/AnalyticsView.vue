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
          <div class="user-menu">
            <span class="user-name">Иван Иванов</span>
            <button @click="handleLogout" class="logout-btn">Выйти</button>
          </div>
        </div>
      </header>

      <!-- Основной контент -->
      <div class="main-content">
        <div class="analytics-container">
          <div class="section-header">
            <h2>Аналитика документов</h2>
          </div>

          <!-- Фильтры -->
          <div class="filters-section">
            <div class="filters-grid">
              <div class="filter-group">
                <label>Год</label>
                <select v-model="selectedYear" class="filter-select" @change="updateStatistics">
                  <option v-for="year in availableYears" :key="year" :value="year">
                    {{ year }}
                  </option>
                  <option value="all">Все годы</option>
                </select>
              </div>
            </div>
          </div>

          <!-- Карточки статистики -->
          <div class="stats-cards">
            <div class="stat-card">
              <div class="stat-icon">📄</div>
              <div class="stat-info">
                <div class="stat-value">{{ totalDocuments }}</div>
                <div class="stat-label">Всего документов</div>
              </div>
            </div>
            <div class="stat-card">
              <div class="stat-icon">📊</div>
              <div class="stat-info">
                <div class="stat-value">{{ yearlyDocuments }}</div>
                <div class="stat-label">
                  {{ selectedYear === 'all' ? 'За все время' : `За ${selectedYear} год` }}
                </div>
              </div>
            </div>
            <div class="stat-card">
              <div class="stat-icon">🔖</div>
              <div class="stat-info">
                <div class="stat-value">{{ documentTypesCount }}</div>
                <div class="stat-label">Типов документов</div>
              </div>
            </div>
            <div class="stat-card">
              <div class="stat-icon">👥</div>
              <div class="stat-info">
                <div class="stat-value">{{ counterpartiesCount }}</div>
                <div class="stat-label">Контрагентов</div>
              </div>
            </div>
          </div>

          <!-- Детальная аналитика -->
          <div class="analytics-detailed">
            <!-- Распределение по типам -->
            <div class="analytics-column">
              <div class="analytics-card">
                <div class="analytics-header">
                  <h3>Распределение по типам</h3>
                </div>
                <div class="analytics-content">
                  <div 
                    v-for="type in typeDistribution" 
                    :key="type.name"
                    class="distribution-item"
                  >
                    <div class="distribution-info">
                      <span class="distribution-name">{{ type.name }}</span>
                      <span class="distribution-count">{{ type.count }}</span>
                    </div>
                    <div class="distribution-bar">
                      <div 
                        class="distribution-bar-fill" 
                        :style="{ width: type.percentage + '%' }"
                      ></div>
                    </div>
                    <span class="distribution-percentage">{{ type.percentage }}%</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Контрагенты -->
            <div class="analytics-column">
              <div class="analytics-card">
                <div class="analytics-header">
                  <h3>Топ контрагентов</h3>
                </div>
                <div class="analytics-content">
                  <div 
                    v-for="counterparty in topCounterparties" 
                    :key="counterparty.name"
                    class="counterparty-item"
                  >
                    <div class="counterparty-info">
                      <span class="counterparty-name">{{ counterparty.name }}</span>
                      <span class="counterparty-count">{{ counterparty.count }}</span>
                    </div>
                    <div class="counterparty-bar">
                      <div 
                        class="counterparty-bar-fill" 
                        :style="{ width: counterparty.percentage + '%' }"
                      ></div>
                    </div>
                    <span class="counterparty-percentage">{{ counterparty.percentage }}%</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Статусы документов -->
            <div class="analytics-column">
              <div class="analytics-card">
                <div class="analytics-header">
                  <h3>Статусы документов</h3>
                </div>
                <div class="analytics-content">
                  <div 
                    v-for="status in statusDistribution" 
                    :key="status.name"
                    class="distribution-item"
                  >
                    <div class="distribution-info">
                      <span class="distribution-name">{{ status.name }}</span>
                      <span class="distribution-count">{{ status.count }}</span>
                    </div>
                    <div class="distribution-bar">
                      <div 
                        class="distribution-bar-fill" 
                        :style="{ width: status.percentage + '%' }"
                      ></div>
                    </div>
                    <span class="distribution-percentage">{{ status.percentage }}%</span>
                  </div>
                </div>
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
  name: 'AnalyticsView',
  data() {
    return {
      selectedYear: new Date().getFullYear(),
      availableYears: [2025, 2024, 2023, 2022],
      documents: [
        {
          id: '264917',
          title: 'Договор поставки',
          filename: 'Договор №154/2024.pdf',
          type: 'Договор поставки',
          counterparty: 'ООО "Ромашка"',
          date: '12.02.2025',
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
          filename: 'Акт №45/2023.pdf',
          type: 'Акт',
          counterparty: 'ООО "Ромашка"',
          date: '15.11.2023',
          status: 'Подписан',
          tags: ['Проект Y', 'Финансовый']
        },
        {
          id: '264920',
          title: 'Договор аренды',
          filename: 'Договор №89/2022.pdf',
          type: 'Договор аренды',
          counterparty: 'ООО "Стройсервис"',
          date: '05.08.2022',
          status: 'Завершен',
          tags: ['Аренда', 'Юридический']
        },
        {
          id: '264921',
          title: 'Счёт на оплату',
          filename: 'Счёт №301.pdf',
          type: 'Счёт',
          counterparty: 'ООО "Ромашка"',
          date: '18.06.2024',
          status: 'Ожидает оплаты',
          tags: ['Финансовый']
        },
        {
          id: '264922',
          title: 'Договор оказания услуг',
          filename: 'Договор №201/2024.pdf',
          type: 'Договор оказания услуг',
          counterparty: 'ООО "ТехноПрофи"',
          date: '10.04.2024',
          status: 'Активен',
          tags: ['Услуги', 'Технический']
        }
      ],
      statistics: {
        total: 0,
        yearly: 0,
        typesCount: 0,
        counterpartiesCount: 0,
        typeDistribution: [],
        topCounterparties: [],
        statusDistribution: []
      }
    }
  },
  computed: {
    totalDocuments() {
      return this.statistics.total;
    },
    yearlyDocuments() {
      return this.statistics.yearly;
    },
    documentTypesCount() {
      return this.statistics.typesCount;
    },
    counterpartiesCount() {
      return this.statistics.counterpartiesCount;
    },
    typeDistribution() {
      return this.statistics.typeDistribution;
    },
    topCounterparties() {
      return this.statistics.topCounterparties;
    },
    statusDistribution() {
      return this.statistics.statusDistribution;
    }
  },
  methods: {
    handleLogout() {
      this.$router.push('/login');
    },
    updateStatistics() {
      const filteredDocs = this.documents.filter(doc => {
        const docYear = new Date(this.parseDate(doc.date)).getFullYear();
        return this.selectedYear === 'all' || docYear === this.selectedYear;
      });

      this.statistics.total = this.documents.length;
      this.statistics.yearly = filteredDocs.length;

      const uniqueTypes = new Set(filteredDocs.map(doc => doc.type));
      const uniqueCounterparties = new Set(filteredDocs.map(doc => doc.counterparty));
      
      this.statistics.typesCount = uniqueTypes.size;
      this.statistics.counterpartiesCount = uniqueCounterparties.size;

      this.calculateTypeDistribution(filteredDocs);
      this.calculateCounterpartyStats(filteredDocs);
      this.calculateStatusDistribution(filteredDocs);
    },
    calculateTypeDistribution(docs) {
      const typeCounts = {};
      
      docs.forEach(doc => {
        typeCounts[doc.type] = (typeCounts[doc.type] || 0) + 1;
      });

      const total = docs.length;
      
      this.statistics.typeDistribution = Object.entries(typeCounts)
        .map(([name, count]) => ({
          name,
          count,
          percentage: total > 0 ? Math.round((count / total) * 100) : 0
        }))
        .sort((a, b) => b.count - a.count);
    },
    calculateCounterpartyStats(docs) {
      const counterpartyCounts = {};
      
      docs.forEach(doc => {
        counterpartyCounts[doc.counterparty] = (counterpartyCounts[doc.counterparty] || 0) + 1;
      });

      const total = docs.length;
      
      this.statistics.topCounterparties = Object.entries(counterpartyCounts)
        .map(([name, count]) => ({
          name,
          count,
          percentage: total > 0 ? Math.round((count / total) * 100) : 0
        }))
        .sort((a, b) => b.count - a.count)
        .slice(0, 5);
    },
    calculateStatusDistribution(docs) {
      const statusCounts = {};
      
      docs.forEach(doc => {
        statusCounts[doc.status] = (statusCounts[doc.status] || 0) + 1;
      });

      const total = docs.length;
      
      this.statistics.statusDistribution = Object.entries(statusCounts)
        .map(([name, count]) => ({
          name,
          count,
          percentage: total > 0 ? Math.round((count / total) * 100) : 0
        }))
        .sort((a, b) => b.count - a.count);
    },
    parseDate(dateString) {
      const [day, month, year] = dateString.split('.');
      return `${year}-${month}-${day}`;
    }
  },
  mounted() {
    this.updateStatistics();
  }
}
</script>

<style scoped src="@/styles/components/AnalyticsView.css"></style>