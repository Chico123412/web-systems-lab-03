<script setup lang="ts">
import { computed, ref } from 'vue'
import userData from '../data/user.json'

defineOptions({
  name: 'UsersList',
})

type Gender = 'male' | 'female'
type GenderFilter = 'all' | Gender
type SortOption = 'default' | 'name-asc' | 'name-desc' | 'age-asc' | 'age-desc'

interface UserName {
  first: string
  last: string
}

interface UserLocation {
  city: string
  country: string
}

interface UserDateOfBirth {
  date: string
  age: number
}

interface User {
  id: number
  gender: Gender
  name: UserName
  location: UserLocation
  email: string
  phone: string
  picture: string
  dob: UserDateOfBirth
  hobbies: string[]
  details: string
}

const users = ref<User[]>(userData.users as User[])

const genderFilter = ref<GenderFilter>('all')
const adultsOnly = ref(false)
const sortOption = ref<SortOption>('default')
const visibleDetails = ref<number[]>([])

const displayedUsers = computed<User[]>(() => {
  let result = [...users.value]

  if (genderFilter.value !== 'all') {
    result = result.filter((user) => user.gender === genderFilter.value)
  }

  if (adultsOnly.value) {
    result = result.filter((user) => user.dob.age >= 18)
  }

  switch (sortOption.value) {
    case 'name-asc':
      result.sort((firstUser, secondUser) =>
          firstUser.name.first.localeCompare(secondUser.name.first, 'uk'),
      )
      break

    case 'name-desc':
      result.sort((firstUser, secondUser) =>
          secondUser.name.first.localeCompare(firstUser.name.first, 'uk'),
      )
      break

    case 'age-asc':
      result.sort((firstUser, secondUser) => firstUser.dob.age - secondUser.dob.age)
      break

    case 'age-desc':
      result.sort((firstUser, secondUser) => secondUser.dob.age - firstUser.dob.age)
      break
  }

  return result
})

function setGenderFilter(filter: GenderFilter): void {
  genderFilter.value = filter
}

function setAdultsFilter(value: boolean): void {
  adultsOnly.value = value
}

function setSortOption(option: SortOption): void {
  sortOption.value = option
}

function toggleDetails(userId: number): void {
  if (visibleDetails.value.includes(userId)) {
    visibleDetails.value = visibleDetails.value.filter((id) => id !== userId)
    return
  }

  visibleDetails.value.push(userId)
}

function resetFilters(): void {
  genderFilter.value = 'all'
  adultsOnly.value = false
  sortOption.value = 'default'
}

function isDetailsVisible(userId: number): boolean {
  return visibleDetails.value.includes(userId)
}
</script>

<template>
  <main class="users-page">
    <header class="page-header">
      <p class="eyebrow">Лабораторна робота №3</p>
      <h1>Список користувачів</h1>
      <p class="subtitle">
        Компоненти, шаблони, реактивність і директиви Vue 3
      </p>
    </header>

    <section class="toolbar">
      <div class="filter-group">
        <span class="filter-title">Стать</span>

        <button
            type="button"
            :class="{ active: genderFilter === 'all' }"
            @click="setGenderFilter('all')"
        >
          Всі
        </button>

        <button
            type="button"
            :class="{ active: genderFilter === 'male' }"
            @click="setGenderFilter('male')"
        >
          Чоловіки
        </button>

        <button
            type="button"
            :class="{ active: genderFilter === 'female' }"
            @click="setGenderFilter('female')"
        >
          Жінки
        </button>
      </div>

      <div class="filter-group">
        <span class="filter-title">Вік</span>

        <button
            type="button"
            :class="{ active: !adultsOnly }"
            @click="setAdultsFilter(false)"
        >
          Всі
        </button>

        <button
            type="button"
            :class="{ active: adultsOnly }"
            @click="setAdultsFilter(true)"
        >
          18+
        </button>
      </div>

      <div class="filter-group">
        <span class="filter-title">Сортування</span>

        <button
            type="button"
            :class="{ active: sortOption === 'name-asc' }"
            @click="setSortOption('name-asc')"
        >
          Ім’я ↑
        </button>

        <button
            type="button"
            :class="{ active: sortOption === 'name-desc' }"
            @click="setSortOption('name-desc')"
        >
          Ім’я ↓
        </button>

        <button
            type="button"
            :class="{ active: sortOption === 'age-asc' }"
            @click="setSortOption('age-asc')"
        >
          Вік ↑
        </button>

        <button
            type="button"
            :class="{ active: sortOption === 'age-desc' }"
            @click="setSortOption('age-desc')"
        >
          Вік ↓
        </button>
      </div>

      <button class="reset-button" type="button" @click="resetFilters">
        Очистити все
      </button>
    </section>

    <p class="results-count">
      Знайдено користувачів: {{ displayedUsers.length }}
    </p>

    <section v-if="displayedUsers.length > 0" class="users-grid">
      <article
          v-for="user in displayedUsers"
          :key="user.id"
          class="user-card"
          :class="{
          minor: user.dob.age < 18,
          young: user.dob.age >= 18 && user.dob.age <= 30,
          adult: user.dob.age >= 31 && user.dob.age <= 50,
          senior: user.dob.age > 50,
        }"
      >
        <img
            v-bind:src="user.picture"
            :alt="`Фото ${user.name.first} ${user.name.last}`"
            class="user-photo"
        />

        <div class="user-content">
          <span class="category">
            {{
              user.dob.age < 18
                  ? 'Minor'
                  : user.dob.age <= 30
                      ? 'Young'
                      : user.dob.age <= 50
                          ? 'Adult'
                          : 'Senior'
            }}
          </span>

          <h2>{{ user.name.first }} {{ user.name.last }}</h2>

          <p>
            <strong>Стать:</strong>
            {{ user.gender === 'male' ? 'Чоловіча' : 'Жіноча' }}
          </p>

          <p v-if="user.dob.age > 18">
            <strong>Вік:</strong> {{ user.dob.age }}
          </p>

          <p><strong>Дата народження:</strong> {{ user.dob.date }}</p>

          <p>
            <strong>Місто:</strong>
            {{ user.location.city }}, {{ user.location.country }}
          </p>

          <p><strong>Email:</strong> {{ user.email }}</p>
          <p><strong>Телефон:</strong> {{ user.phone }}</p>

          <div class="hobbies">
            <strong>Хобі:</strong>

            <ul>
              <li v-for="hobby in user.hobbies" :key="hobby">
                {{ hobby }}
              </li>
            </ul>
          </div>

          <button
              class="details-button"
              type="button"
              @click="toggleDetails(user.id)"
          >
            {{
              isDetailsVisible(user.id)
                  ? 'Приховати інформацію'
                  : 'Показати інформацію'
            }}
          </button>

          <p v-show="isDetailsVisible(user.id)" class="details">
            {{ user.details }}
          </p>
        </div>
      </article>
    </section>

    <p v-else class="empty-message">Список юзерів пустий</p>
  </main>
</template>

<style scoped>
.users-page {
  width: min(1200px, calc(100% - 32px));
  margin: 0 auto;
  padding: 48px 0 64px;
}

.page-header {
  margin-bottom: 28px;
  text-align: center;
}

.eyebrow {
  margin: 0 0 8px;
  color: #2563eb;
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

h1 {
  margin: 0;
  color: #172033;
  font-size: clamp(32px, 5vw, 52px);
}

.subtitle {
  margin: 12px 0 0;
  color: #64748b;
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  align-items: end;
  padding: 20px;
  border: 1px solid #dbe3ef;
  border-radius: 18px;
  background: #ffffff;
  box-shadow: 0 10px 30px rgb(15 23 42 / 8%);
}

.filter-group {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.filter-title {
  flex-basis: 100%;
  color: #475569;
  font-size: 13px;
  font-weight: 700;
}

button {
  padding: 9px 14px;
  border: 1px solid #cbd5e1;
  border-radius: 9px;
  background: #ffffff;
  color: #334155;
  font: inherit;
  font-weight: 600;
  cursor: pointer;
  transition:
      background 0.2s,
      color 0.2s,
      transform 0.2s;
}

button:hover {
  transform: translateY(-1px);
  border-color: #2563eb;
}

button.active {
  border-color: #2563eb;
  background: #2563eb;
  color: #ffffff;
}

.reset-button {
  margin-left: auto;
  border-color: #ef4444;
  color: #dc2626;
}

.results-count {
  margin: 24px 0 16px;
  color: #475569;
  font-weight: 700;
}

.users-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(310px, 1fr));
  gap: 20px;
}

.user-card {
  overflow: hidden;
  border: 2px solid transparent;
  border-radius: 20px;
  background: #ffffff;
  box-shadow: 0 12px 30px rgb(15 23 42 / 10%);
}

.user-card.minor {
  border-color: #f59e0b;
}

.user-card.young {
  border-color: #22c55e;
}

.user-card.adult {
  border-color: #3b82f6;
}

.user-card.senior {
  border-color: #8b5cf6;
}

.user-photo {
  width: 100%;
  height: 240px;
  object-fit: cover;
}

.user-content {
  padding: 20px;
}

.category {
  display: inline-block;
  margin-bottom: 10px;
  padding: 5px 10px;
  border-radius: 999px;
  background: #eef2ff;
  color: #4338ca;
  font-size: 12px;
  font-weight: 800;
  text-transform: uppercase;
}

.user-content h2 {
  margin: 0 0 14px;
  color: #172033;
}

.user-content p {
  margin: 8px 0;
  overflow-wrap: anywhere;
  color: #475569;
}

.hobbies {
  margin-top: 14px;
  color: #475569;
}

.hobbies ul {
  display: flex;
  flex-wrap: wrap;
  gap: 7px;
  margin: 8px 0 0;
  padding: 0;
  list-style: none;
}

.hobbies li {
  padding: 5px 9px;
  border-radius: 8px;
  background: #f1f5f9;
  font-size: 14px;
}

.details-button {
  width: 100%;
  margin-top: 18px;
  border-color: #2563eb;
  color: #2563eb;
}

.details {
  margin-top: 14px !important;
  padding: 12px;
  border-radius: 10px;
  background: #f8fafc;
}

.empty-message {
  padding: 48px;
  border-radius: 18px;
  background: #ffffff;
  color: #64748b;
  text-align: center;
  font-size: 20px;
  font-weight: 700;
}

@media (max-width: 700px) {
  .users-page {
    width: min(100% - 20px, 1200px);
    padding-top: 28px;
  }

  .toolbar {
    align-items: stretch;
  }

  .filter-group {
    width: 100%;
  }

  .reset-button {
    width: 100%;
    margin-left: 0;
  }
}
</style>