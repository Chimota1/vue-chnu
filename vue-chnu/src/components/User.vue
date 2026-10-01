<script setup lang="ts">
import { ref, computed } from 'vue'
import userData from '../user.json'

// Форматування дати
const formatDate = (dateString: string) => {
  const date = new Date(dateString)
  return `${date.getDate().toString().padStart(2, '0')}.${(date.getMonth() + 1).toString().padStart(2, '0')}.${date.getFullYear()}`
}

// Стан для директиви v-show
const showDetails = ref(false)

// Обчислювана властивість для прив'язки стилів :class залежно від віку
const ageClass = computed(() => {
  const age = userData.dob.age
  return {
    'minor': age < 18,
    'young': age >= 18 && age <= 30,
    'adult': age >= 31 && age <= 50,
    'senior': age > 50
  }
})
</script>

<template>
  <div class="resume-card" :class="ageClass">
    <!-- ЛІВА КОЛОНКА -->
    <div class="sidebar">
      <img :src="userData.picture" alt="Avatar" class="avatar" />

      <h1 class="name">{{ userData.name.title }} {{ userData.name.first }} {{ userData.name.last }}</h1>

      <div class="quick-tags">
        <span class="tag">👩 {{ userData.gender === 'female' ? 'Female' : 'Male' }}</span>
        <span class="tag" v-if="userData.dob.age > 18">📅 {{ userData.dob.age }} years</span>
      </div>

      <div class="contact-list">
        <div class="contact-item">
          <span class="icon">📍</span>
          <span>{{ userData.location.city }}, {{ userData.location.state }}, {{ userData.location.country }}</span>
        </div>
        <div class="contact-item">
          <span class="icon">✉️</span>
          <a :href="`mailto:${userData.email}`">{{ userData.email }}</a>
        </div>
        <div class="contact-item">
          <span class="icon">📞</span>
          <span>{{ userData.phone }}</span>
        </div>
        <div class="contact-item">
          <span class="icon">📱</span>
          <span>{{ userData.cell }}</span>
        </div>
      </div>
    </div>

    <!-- ПРАВА КОЛОНКА -->
    <div class="content">

      <!-- Блок About me -->
      <div class="content-section clickable" @click="showDetails = !showDetails">
        <h3 class="section-title">
          <span>👤 About me</span>
          <span class="chevron">{{ showDetails ? '▲' : '▼' }}</span>
        </h3>
      </div>
      <div class="details-text" v-show="showDetails">
        {{ userData.details }}
      </div>

      <!-- Блок Personal Information -->
      <div class="content-section">
        <h3 class="section-title">📄 Personal Information</h3>
        <div class="info-grid">
          <div class="label">Full name</div>
          <div class="value">{{ userData.name.title }} {{ userData.name.first }} {{ userData.name.last }}</div>

          <div class="label">Gender</div>
          <div class="value capitalize">{{ userData.gender }}</div>

          <div class="label">Date of birth</div>
          <div class="value">{{ formatDate(userData.dob.date) }} (age {{ userData.dob.age }})</div>

          <div class="label">Email</div>
          <div class="value"><a :href="`mailto:${userData.email}`">{{ userData.email }}</a></div>

          <div class="label">Phone</div>
          <div class="value">{{ userData.phone }}</div>

          <div class="label">Cell</div>
          <div class="value">{{ userData.cell }}</div>
        </div>
      </div>

      <!-- Блок Location -->
      <div class="content-section">
        <h3 class="section-title">📍 Location</h3>
        <div class="info-grid">
          <div class="label">Street</div>
          <div class="value">{{ userData.location.street.number }} {{ userData.location.street.name }}</div>

          <div class="label">City</div>
          <div class="value">{{ userData.location.city }}</div>

          <div class="label">State</div>
          <div class="value">{{ userData.location.state }}</div>

          <div class="label">Country</div>
          <div class="value">{{ userData.location.country }}</div>

          <div class="label">Postcode</div>
          <div class="value">{{ userData.location.postcode }}</div>

          <div class="label">Timezone</div>
          <div class="value">{{ userData.location.timezone.offset }} ({{ userData.location.timezone.description }})</div>
        </div>
      </div>

      <!-- Блок Hobbies -->
      <div class="content-section">
        <h3 class="section-title">⭐ Hobbies</h3>
        <div class="hobbies-list">
          <span
            v-for="(hobby, index) in userData.hobbies"
            :key="index"
            class="hobby-chip"
          >
            {{ hobby }}
          </span>
        </div>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* Глобальний контейнер картки */
.resume-card {
  display: flex;
  flex-direction: row;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
  max-width: 750px; /* Зменшено з 950px */
  margin: 20px auto; /* Менший зовнішній відступ */
  overflow: hidden;
  border: 1px solid #eaeaea;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  border-top: 5px solid transparent;
}

/* Динамічні класи віку */
.resume-card.minor { border-top-color: #f1c40f; }
.resume-card.young { border-top-color: #2ecc71; }
.resume-card.adult { border-top-color: #3498db; }
.resume-card.senior { border-top-color: #9b59b6; }

/* ЛІВА КОЛОНКА */
.sidebar {
  width: 260px; /* Зменшено з 320px */
  padding: 24px 20px; /* Зменшено відступи */
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
  font-size: 20px; /* Зменшено шрифт */
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
  gap: 12px; /* Зменшено проміжок між контактами */
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 13px;
  color: #4a5568;
}

.contact-item a {
  color: #4a5568;
  text-decoration: none;
}
.contact-item a:hover {
  text-decoration: underline;
}

/* ПРАВА КОЛОНКА */
.content {
  flex: 1;
  padding: 24px; /* Зменшено відступи */
  display: flex;
  flex-direction: column;
  gap: 20px; /* Зменшено проміжок між секціями */
}

.content-section {
  border: 1px solid #eaeaea;
  border-radius: 8px;
  padding: 16px; /* Зменшено відступи всередині секцій */
}

.clickable {
  cursor: pointer;
  background-color: #f8f9fa;
  transition: background 0.2s;
}
.clickable:hover {
  background-color: #f1f3f5;
}

.section-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 0;
  font-size: 15px; /* Зменшено шрифт заголовків секцій */
  font-weight: 600;
  color: #2d3748;
}

.details-text {
  padding: 0 16px 16px 16px;
  color: #4a5568;
  font-size: 13px;
  line-height: 1.5;
}

/* Таблична сітка для інформації */
.info-grid {
  display: grid;
  grid-template-columns: 120px 1fr; /* Зменшено ширину першої колонки */
  row-gap: 10px; /* Зменшено міжрядковий інтервал */
  margin-top: 14px;
  font-size: 13px;
}

.label {
  color: #718096;
}

.value {
  color: #2d3748;
  font-weight: 500;
}
.value a {
  color: #3182ce;
  text-decoration: none;
}

.capitalize {
  text-transform: capitalize;
}

/* Хобі у вигляді бейджів */
.hobbies-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 14px;
}

.hobby-chip {
  background-color: #ebf4ff;
  color: #3182ce;
  padding: 4px 12px; /* Зроблено компактніші бейджі */
  border-radius: 16px;
  font-size: 12px;
  font-weight: 500;
}

/* Адаптивність для менших екранів */
@media (max-width: 650px) {
  .resume-card {
    flex-direction: column;
  }
  .sidebar {
    width: auto;
    border-right: none;
    border-bottom: 1px solid #eaeaea;
  }
  .info-grid {
    grid-template-columns: 1fr;
    row-gap: 6px;
  }
}
</style>
