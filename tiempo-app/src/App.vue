<template>
  <div class="app" :style="backgroundStyle">
    <div class="weather-card">
      <input
        v-model="city"
        @keyup.enter="getWeather"
        placeholder="Escribe una ciudad..."
        class="city-input"
      />
      <div v-if="loading" class="loading">Cargando clima...</div>
      <div v-else-if="weather">
        <h1>{{ weather.name }}</h1>
        <h2>{{ Math.round(weather.main.temp) }}°C</h2>
        <p>{{ weather.weather[0].description }}</p>
      </div>
      <div v-else class="loading">Introduce una ciudad y presiona Enter</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const city = ref('')
const weather = ref(null)
const loading = ref(false)

const apiKey = import.meta.env.VITE_WEATHER_API_KEY

const getWeather = async () => {
  if (!city.value) return
  loading.value = true
  try {
    const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(
      city.value,
    )}&appid=${apiKey}&units=metric&lang=es`
    const res = await fetch(url)
    const data = await res.json()
    if (data.cod === 200) {
      weather.value = data
    } else {
      alert(data.message)
    }
  } catch (err) {
    console.error(err)
    alert('Error al obtener el clima')
  } finally {
    loading.value = false
  }
}

// Fondo dinámico según clima
const backgroundStyle = computed(() => {
  if (!weather.value) return { background: 'linear-gradient(135deg, #89f7fe, #66a6ff)' }
  const main = weather.value.weather[0].main
  if (main.includes('Cloud')) return { background: 'linear-gradient(135deg, #757f9a, #d7dde8)' }
  if (main.includes('Rain')) return { background: 'linear-gradient(135deg, #667db6, #0082c8)' }
  if (main.includes('Clear')) return { background: 'linear-gradient(135deg, #f7971e, #ffd200)' }
  if (main.includes('Snow')) return { background: 'linear-gradient(135deg, #83a4d4, #b6fbff)' }
  return { background: 'linear-gradient(135deg, #89f7fe, #66a6ff)' }
})
</script>

<style>
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}

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
  min-width: 280px;
}

.city-input {
  width: 80%;
  padding: 0.5rem;
  border-radius: 1rem;
  border: none;
  margin-bottom: 1rem;
  text-align: center;
  font-size: 1rem;
}

.city-input:focus {
  outline: none;
  box-shadow: 0 0 5px rgba(255, 255, 255, 0.7);
}

.loading {
  font-size: 1rem;
}
</style>
