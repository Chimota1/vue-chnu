<script setup lang="ts">
import { ref, computed } from 'vue'
import usersData from '../user.json'

// --- СТАНИ ДЛЯ ФІЛЬТРІВ ТА СОРТУВАННЯ ---
const genderFilter = ref('Всі') // 'Всі', 'male', 'female'
const ageFilter = ref('Всі') // 'Всі', '18+'
const sortBy = ref('default') // 'default', 'nameAsc', 'nameDesc', 'ageAsc', 'ageDesc'

// Стан для директиви v-show (зберігає id юзерів, у яких відкриті деталі)
const openDetails = ref<Record<number, boolean>>({})

// Метод для перемикання деталей конкретного юзера
const toggleDetails = (id: number) => {
  openDetails.value[id] = !openDetails.value[id]
}

// Форматування дати
const formatDate = (dateString: string) => {
  const date = new Date(dateString)
  return `${date.getDate().toString().padStart(2, '0')}.${(date.getMonth() + 1).toString().padStart(2, '0')}.${date.getFullYear()}`
}

// Метод для динамічного класу віку (перенесено з computed, бо тепер залежить від юзера)
const getAgeClass = (age: number) => {
  if (age < 18) return 'minor'
  if (age <= 30) return 'young'
  if (age <= 50) return 'adult'
  return 'senior'
}

// Метод скидання всіх фільтрів
const resetFilters = () => {
  genderFilter.value = 'Всі'
  ageFilter.value = 'Всі'
  sortBy.value = 'default'
}

// --- ГОЛОВНА ЛОГІКА ФІЛЬТРАЦІЇ ТА СОРТУВАННЯ ---
const processedUsers = computed(() => {
  // Робимо копію масиву, щоб не мутувати оригінальні дані
  let result = [...usersData]

  // 1. Фільтр за статтю
  if (genderFilter.value !== 'Всі') {
    result = result.filter(user => user.gender === genderFilter.value)
  }

  // 2. Фільтр за віком
  if (ageFilter.value === '18+') {
    result = result.filter(user => user.dob.age >= 18)
  }

  // 3. Сортування
  if (sortBy.value === 'nameAsc') {
    result.sort((a, b) => a.name.first.localeCompare(b.name.first))
  } else if (sortBy.value === 'nameDesc') {
    result.sort((a, b) => b.name.first.localeCompare(a.name.first))
  } else if (sortBy.value === 'ageAsc') {
    result.sort((a, b) => a.dob.age - b.dob.age)
  } else if (sortBy.value === 'ageDesc') {
    result.sort((a, b) => b.dob.age - a.dob.age)
  }

  return result
})
</script>

<template>
  <div class="users-page">

    <!-- ТУЛБАР (Панель керування) -->
    <div class="toolbar">
      <div class="toolbar-group">
        <span class="toolbar-label">Стать:</span>
        <button :class="{ active: genderFilter === 'Всі' }" @click="genderFilter = 'Всі'">Всі</button>
        <button :class="{ active: genderFilter === 'male' }" @click="genderFilter = 'male'">Чоловіки</button>
        <button :class="{ active: genderFilter === 'female' }" @click="genderFilter = 'female'">Жінки</button>
      </div>

      <div class="toolbar-group">
        <span class="toolbar-label">Вік:</span>
        <button :class="{ active: ageFilter === 'Всі' }" @click="ageFilter = 'Всі'">Всі</button>
        <button :class="{ active: ageFilter === '18+' }" @click="ageFilter = '18+'">18 +</button>
      </div>

      <div class="toolbar-group">
        <span class="toolbar-label">Сортування:</span>
        <button :class="{ active: sortBy === 'nameAsc' }" @click="sortBy = 'nameAsc'">Ім’я ↑</button>
        <button :class="{ active: sortBy === 'nameDesc' }" @click="sortBy = 'nameDesc'">Ім’я ↓</button>
        <button :class="{ active: sortBy === 'ageAsc' }" @click="sortBy = 'ageAsc'">Вік ↑</button>
        <button :class="{ active: sortBy === 'ageDesc' }" @click="sortBy = 'ageDesc'">Вік ↓</button>
      </div>

      <button class="reset-btn" @click="resetFilters">Очистити все</button>
    </div>

    <!-- ПУСТИЙ СТАН -->
    <div v-if="processedUsers.length === 0" class="empty-state">
      <h2>Список юзерів пустий 🕵️‍♂️</h2>
      <p>За цими фільтрами нікого не знайдено.</p>
    </div>

    <!-- СПИСОК ЮЗЕРІВ -->
    <div v-else class="users-list">
      <!-- v-for проходить по ВЖЕ відфільтрованому і відсортованому масиву -->
      <div
        v-for="user in processedUsers"
        :key="user.id"
        class="resume-card"
        :class="getAgeClass(user.dob.age)"
      >
        <!-- ЛІВА КОЛОНКА -->
        <div class="sidebar">
          <!-- Динамічний атрибут alt на основі імені -->
          <img :src="user.picture" :alt="`${user.name.first} ${user.name.last}`" class="avatar" />

          <h1 class="name">{{ user.name.title }} {{ user.name.first }} {{ user.name.last }}</h1>

          <div class="quick-tags">
            <span class="tag">👩 {{ user.gender === 'female' ? 'Female' : 'Male' }}</span>
            <span class="tag" v-if="user.dob.age > 18">📅 {{ user.dob.age }} years</span>
          </div>

          <div class="contact-list">
            <div class="contact-item">
              <span class="icon">📍</span>
              <span>{{ user.location.city }}, {{ user.location.state }}, {{ user.location.country }}</span>
            </div>
            <div class="contact-item">
              <span class="icon">✉️</span>
              <a :href="`mailto:${user.email}`">{{ user.email }}</a>
            </div>
            <div class="contact-item">
              <span class="icon">📞</span>
              <span>{{ user.phone }}</span>
            </div>
          </div>
        </div>

        <!-- ПРАВА КОЛОНКА -->
        <div class="content">
          <!-- Блок About me (Деталі) -->
          <div class="content-section clickable" @click="toggleDetails(user.id)">
            <h3 class="section-title">
              <span>👤 About me</span>
              <span class="chevron">{{ openDetails[user.id] ? '▲' : '▼' }}</span>
            </h3>
          </div>
          <div class="details-text" v-show="openDetails[user.id]">
            {{ user.details }}
          </div>

          <!-- Блок Personal Information -->
          <div class="content-section">
            <h3 class="section-title">📄 Personal Information</h3>
            <div class="info-grid">
              <div class="label">Date of birth</div>
              <div class="value">{{ formatDate(user.dob.date) }} (age {{ user.dob.age }})</div>

              <div class="label">Cell</div>
              <div class="value">{{ user.cell }}</div>
            </div>
          </div>

          <!-- Блок Hobbies -->
          <div class="content-section">
            <h3 class="section-title">⭐ Hobbies</h3>
            <div class="hobbies-list">
              <span v-for="(hobby, index) in user.hobbies" :key="index" class="hobby-chip">
                {{ hobby }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* СТИЛІ ТУЛБАРУ */
.users-page {
  max-width: 800px;
  margin: 0 auto;
}

.toolbar {
  background: #ffffff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.05);
  margin-bottom: 30px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  border: 1px solid #eaeaea;
}

.toolbar-group {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

.toolbar-label {
  font-weight: 600;
  color: #4a5568;
  width: 100px;
}

.toolbar button {
  background: #f1f3f5;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  color: #4a5568;
  font-weight: 500;
  transition: all 0.2s;
}

.toolbar button:hover {
  background: #e2e8f0;
}

.toolbar button.active {
  background: #3182ce;
  color: white;
}

.toolbar .reset-btn {
  background: #e53e3e;
  color: white;
  margin-top: 10px;
  align-self: flex-start;
}
.toolbar .reset-btn:hover { background: #c53030; }

.empty-state {
  text-align: center;
  padding: 60px 20px;
  background: #ffffff;
  border-radius: 12px;
  color: #718096;
  border: 1px dashed #cbd5e0;
}

.users-list {
  display: flex;
  flex-direction: column;
  gap: 25px;
}

/* СТИЛІ КАРТКИ (трохи скорочені для компактності списку) */
.resume-card {
  display: flex;
  flex-direction: row;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  overflow: hidden;
  border: 1px solid #eaeaea;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  border-top: 5px solid transparent;
}

.resume-card.minor { border-top-color: #f1c40f; }
.resume-card.young { border-top-color: #2ecc71; }
.resume-card.adult { border-top-color: #3498db; }
.resume-card.senior { border-top-color: #9b59b6; }

.sidebar {
  width: 240px;
  padding: 24px 20px;
  border-right: 1px solid #eaeaea;
  background-color: #fafbfc;
}

.avatar {
  width: 100%;
  aspect-ratio: 1;
  border-radius: 10px;
  object-fit: cover;
  margin-bottom: 16px;
}

.name {
  font-size: 18px;
  font-weight: 700;
  color: #1a202c;
  margin: 0 0 10px 0;
}

.quick-tags {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  color: #718096;
  font-size: 13px;
}

.contact-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #4a5568;
}

.contact-item a { color: #4a5568; text-decoration: none; }
.contact-item a:hover { text-decoration: underline; }

.content {
  flex: 1;
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.content-section {
  border: 1px solid #eaeaea;
  border-radius: 8px;
  padding: 12px 16px;
}

.clickable {
  cursor: pointer;
  background-color: #f8f9fa;
  transition: background 0.2s;
}
.clickable:hover { background-color: #f1f3f5; }

.section-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0;
  font-size: 14px;
  font-weight: 600;
  color: #2d3748;
}

.details-text {
  padding: 0 16px 16px 16px;
  color: #4a5568;
  font-size: 13px;
  line-height: 1.5;
}

.info-grid {
  display: grid;
  grid-template-columns: 100px 1fr;
  row-gap: 8px;
  margin-top: 10px;
  font-size: 13px;
}

.label { color: #718096; }
.value { color: #2d3748; font-weight: 500; }

.hobbies-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}

.hobby-chip {
  background-color: #ebf4ff;
  color: #3182ce;
  padding: 4px 10px;
  border-radius: 16px;
  font-size: 12px;
  font-weight: 500;
}

@media (max-width: 650px) {
  .resume-card { flex-direction: column; }
  .sidebar { width: auto; border-right: none; border-bottom: 1px solid #eaeaea; }
}
</style>
