<script setup lang="ts">
// Імпортуємо ваш JSON-файл (зверніть увагу на велику літеру У, якщо файл названо User.json)
import userData from './user.json'

// Функція для форматування дати
const formatDate = (dateString: string) => {
  return new Date(dateString).toLocaleDateString('uk-UA')
}
</script>

<template>
  <div class="user-card">
    <div class="user-image">
      <img :src="userData.picture" :alt="`${userData.name.first} ${userData.name.last}`" />
    </div>

    <div class="user-info">
      <h2>{{ userData.name.title }} {{ userData.name.first }} {{ userData.name.last }}</h2>

      <p><strong>Стать:</strong> {{ userData.gender === 'female' ? 'Жінка' : 'Чоловік' }}</p>
      <p><strong>Вік:</strong> {{ userData.dob.age }} років ({{ formatDate(userData.dob.date) }})</p>
      <p><strong>Адреса:</strong> {{ userData.location.country }}, м. {{ userData.location.city }}, {{ userData.location.street.name }} {{ userData.location.street.number }}</p>
      <p><strong>Email:</strong> <a :href="`mailto:${userData.email}`">{{ userData.email }}</a></p>
      <p><strong>Телефон:</strong> <a :href="`tel:${userData.phone}`">{{ userData.phone }}</a></p>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  display: flex;
  align-items: center;
  gap: 24px;
  background-color: #ffffff;
  border-radius: 16px;
  padding: 24px;
  max-width: 550px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  margin: 40px auto;
  border: 1px solid #e0e0e0;
}
.user-image img {
  width: 130px;
  height: 130px;
  border-radius: 50%;
  object-fit: cover;
  border: 4px solid #42b883;
}
.user-info h2 {
  margin: 0 0 15px 0;
  color: #2c3e50;
}
.user-info p {
  margin: 8px 0;
  color: #555;
}
</style>
