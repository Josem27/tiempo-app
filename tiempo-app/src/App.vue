<template>
  <div class="app" :style="backgroundStyle">
    <div class="weather-card" v-if="weather">
      <h1>{{ weather.name }}</h1>
      <h2>{{ Math.round(weather.main.temp) }}°C</h2>
      <p>{{ weather.weather[0].description }}</p>
    </div>
    <div v-else class="loading">
      <p>Cargando clima...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const weather = ref(null)

const fetchWeather = async () => {
  const lat = 40.4168
  const lon = -3.7038
  const apiKey = import.meta.env.VITE_WEATHER_API_KEY
  const url = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric&lang=es`
  const res = await fetch(url)
  weather.value = await res.json()
}

onMounted(() => {
  fetchWeather()
})

const backgroundStyle = computed(() => {
  if (!weather.value) return {}
  const main = weather.value.weather[0].main
  if (main.includes('Cloud')) return { background: 'linear-gradient(135deg, #757f9a, #d7dde8)' }
  if (main.includes('Rain')) return { background: 'linear-gradient(135deg, #667db6, #0082c8)' }
  if (main.includes('Clear')) return { background: 'linear-gradient(135deg, #f7971e, #ffd200)' }
  if (main.includes('Snow')) return { background: 'linear-gradient(135deg, #83a4d4, #b6fbff)' }
  return { background: 'linear-gradient(135deg, #89f7fe, #66a6ff)' }
})
</script>

<style>
.app {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background 0.5s ease;
}
.weather-card {
  background: rgba(255, 255, 255, 0.15);
  padding: 2rem 3rem;
  border-radius: 2rem;
  backdrop-filter: blur(10px);
  text-align: center;
  color: #fff;
}
.loading {
  color: #fff;
  font-size: 1.2rem;
}
</style>
