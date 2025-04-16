<script setup>
import { ref, onMounted } from 'vue'
import { useToast } from 'vue-toastification'
import clearGif from '@/assets/clear.gif'
import cloudGif from '@/assets/cloud.gif'
import rainGif from '@/assets/rain.gif'
import sunGif from '@/assets/sun.gif'
import thunderGif from '@/assets/thunder.gif'
import snowGif from '@/assets/snow.gif'

const toast = useToast()

const query = ref('')
const temp = ref('C')
const weather = ref(null)
const weatherTemp = ref(0)
const icon = ref('')
const des = ref('')
const main = ref('')
const forecast = ref([])
const name = ref('')
const favoriteCity = ref('')
const code = ref('')
const imgsrc = ref('')
const weatherIcon = ref('')
const loading = ref(false)

const iconMap = {
  Clear: clearGif,
  Clouds: cloudGif,
  Rain: rainGif,
  Drizzle: sunGif,
  Thunderstorm: thunderGif,
  Snow: snowGif,
}

const searchQuery = async (storedCity = '') => {
  const apiKey = '4e519145b38cf420f054cadc4c815a9f'
  const city = query.value || storedCity

  if (!city) {
    toast.error('Please enter a city')
    return
  }

  const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apiKey}&units=metric`
  loading.value = true

  try {
    const response = await fetch(forecastUrl)
    if (!response.ok) throw new Error('City not found')

    const data = await response.json()
    weather.value = data

    name.value = data.city.name
    code.value = data.city.country
    imgsrc.value = `https://flagsapi.com/${code.value.toUpperCase()}/flat/64.png`
    weatherTemp.value = Math.round(data.list[0].main.temp)
    icon.value = `https://openweathermap.org/img/wn/${data.list[0].weather[0].icon}@2x.png`
    main.value = data.list[0].weather[0].main
    weatherIcon.value = iconMap[main.value] || iconMap['Snow']
    des.value = data.list[0].weather[0].description

    forecast.value = data.list.filter((item) => item.dt_txt.includes('12:00:00')).slice(0, 5)
    query.value = ''
  } catch (error) {
    toast.error(error.message || 'An error occurred')
    weather.value = null
    forecast.value = []
  } finally {
    loading.value = false
  }
}

const toFahrenheit = (celsius) => ((celsius * 9) / 5 + 32).toFixed(0)

const addCity = () => {
  if (!name.value) return
  favoriteCity.value = name.value
  localStorage.setItem('favoriteCity', favoriteCity.value)
  toast.success(`${favoriteCity.value} added as favorite city`)
}

const formatDate = (datetime) => {
  const options = { weekday: 'short', month: 'short', day: 'numeric' }
  return new Date(datetime).toLocaleDateString(undefined, options)
}

onMounted(() => {
  const storedCity = localStorage.getItem('favoriteCity')
  if (storedCity) searchQuery(storedCity)
})
</script>

<template>
  <div class="container">
    <div class="top">
      <h1>Get Accurate Weather Results on Forecast Today</h1>
      <form @submit.prevent="searchQuery()">
        <input type="search" v-model="query" placeholder="Enter city name" />
        <div class="radio-group">
          <label><input type="radio" v-model="temp" value="F" /> F</label>
          <label><input type="radio" v-model="temp" value="C" /> C</label>
        </div>
        <input type="submit" value="Search" />
      </form>
    </div>

    <div v-if="loading" class="loading">Loading city info...</div>

    <div v-else-if="weather" class="bottom">
      <img :src="weatherIcon" alt="Weather Image" class="weather-icon" loading="lazy" />
      <div class="temp">
        {{ temp === 'C' ? weatherTemp : toFahrenheit(weatherTemp) }} °{{ temp }}
      </div>
      <div class="main">{{ main }}</div>
      <div class="location">
        {{ name }} ,
        <img :src="imgsrc" alt="flag" class="flags" />
      </div>
      <button id="fav" @click="addCity">Fav</button>

      <div v-if="forecast.length" class="forecast">
        <h3>5-Day Forecast</h3>
        <transition-group name="fade" tag="div">
          <div v-for="(day, index) in forecast" :key="index" class="days">
            <small>{{ formatDate(day.dt_txt) }}</small>
            <small>
              Temp:
              {{
                temp === 'C' ? Math.round(day.main.temp) : toFahrenheit(Math.round(day.main.temp))
              }}
              °{{ temp }}
            </small>
            <div class="holder">
              <img
                :src="`https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`"
                :alt="day.weather[0].description"
              />
              <small>{{ day.weather[0].description }}</small>
            </div>
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');

.container {
  margin: 2rem auto;
  max-width: 600px;
  background: linear-gradient(to bottom right, #e0f2ff, #cfd9df);
  padding: 2rem;
  border-radius: 1rem;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  font-family: 'Inter', sans-serif;
}

.top h1 {
  font-size: 1.5rem;
  text-align: center;
  margin-bottom: 1rem;
  color: #1a202c;
}

form input[type='search'],
form input[type='submit'] {
  width: 100%;
  padding: 0.5rem;
  margin-bottom: 1rem;
  border-radius: 0.5rem;
}

form input[type='search'] {
  border: 1px solid #ccc;
}

.radio-group {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

form input[type='submit'] {
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
}

form input[type='submit']:hover {
  background-color: #0056b3;
}

@keyframes round {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.bottom {
  text-align: center;
  margin-top: 2rem;
  animation: slideIn 1s ease forwards;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(40px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.weather-icon {
  width: 100px;
  height: 100px;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

.temp {
  font-size: 2.5rem;
  font-weight: 600;
  color: #1a202c;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 1rem;
  padding: 1rem 2rem;
  margin: 1rem auto;
  width: fit-content;
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.2);
}

.main {
  font-size: 1.25rem;
  text-transform: capitalize;
  color: #4a5568;
  margin: 0.5rem 0;
}

.location {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  font-size: 1.1rem;
  margin-bottom: 1rem;
  color: #2d3748;
  font-style: italic;
}

#fav {
  padding: 0.5rem 1rem;
  background-color: #f6ad55;
  color: white;
  border: none;
  border-radius: 0.5rem;
  cursor: pointer;
}

#fav:hover {
  background-color: #ed8936;
}

.forecast {
  margin-top: 1rem;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.days {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: center;
  gap: 1rem;
  padding: 0.5rem;
  margin: 1rem 0;
  background: lightblue;
  border-radius: 0.75rem;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.days img {
  width: 40px;
  height: 40px;
}

.holder {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.flags {
  width: 2rem;
  height: 1.5rem;
  border-radius: 0.25rem;
}
.loading {
  text-align: center;
  font-weight: 600;
  font-size: 2.5rem;
  color: #1a202c;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 1rem;
  padding: 1rem 2rem;
  margin: 1rem auto;
  width: fit-content;
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.2);
}
</style>
