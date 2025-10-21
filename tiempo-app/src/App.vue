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
        <img
          :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`"
          alt="icono clima"
          class="weather-icon"
        />
        <h1>{{ weather.name }}, {{ weather.sys.country }}</h1>
        <h2>{{ Math.round(weather.main.temp) }}°C</h2>
        <p>{{ capitalize(weather.weather[0].description) }}</p>
      </div>
      <div v-else class="loading">Introduce una ciudad y presiona Enter</div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

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

// Función para capitalizar la primera letra de la descripción
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1)

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
  background: rgba(255, 255, 255, 0.2);
  padding: 2rem 3rem;
  border-radius: 2rem;
  backdrop-filter: blur(15px);
  text-align: center;
  color: #fff;
  min-width: 280px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.2);
  transition: transform 0.2s ease;
}

.weather-card:hover {
  transform: translateY(-5px);
}

.weather-icon {
  width: 80px;
  height: 80px;
  margin-bottom: 0.5rem;
}

.city-input {
  width: 80%;
  padding: 0.5rem;
  border-radius: 1rem;
  border: none;
  margin-bottom: 1rem;
  text-align: center;
  font-size: 1rem;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
}

.city-input:focus {
  outline: none;
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.7);
}

.loading {
  font-size: 1rem;
}
</style>
