<template>
  <div class="app" :style="backgroundStyle">
    <div class="weather-container">
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

      <transition-group name="forecast" tag="div" class="forecast" v-if="forecast3.length">
        <div
          class="forecast-card"
          v-for="(day, i) in forecast3"
          :key="day.dt"
          :style="{ transitionDelay: i * 0.1 + 's' }"
        >
          <p>{{ day.date }}</p>
          <img :src="`https://openweathermap.org/img/wn/${day.icon}@2x.png`" alt="icono clima" />
          <p>{{ Math.round(day.temp) }}°C</p>
          <p>{{ capitalize(day.description) }}</p>
        </div>
      </transition-group>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const city = ref('')
const weather = ref(null)
const loading = ref(false)
const forecast3 = ref([])

const apiKey = import.meta.env.VITE_WEATHER_API_KEY
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1)

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
      await getForecast(city.value)
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

const getForecast = async (cityName) => {
  try {
    const url = `https://api.openweathermap.org/data/2.5/forecast?q=${encodeURIComponent(
      cityName,
    )}&appid=${apiKey}&units=metric&lang=es`
    const res = await fetch(url)
    const data = await res.json()
    if (data.cod === '200') {
      const days = []
      const today = new Date().getDate()
      const dates = []
      data.list.forEach((item) => {
        const d = new Date(item.dt_txt)
        const dayNum = d.getDate()
        if (!dates.includes(dayNum) && d.getHours() === 12 && dayNum !== today) {
          dates.push(dayNum)
          days.push({
            dt: item.dt,
            date: d.toLocaleDateString('es-ES', { weekday: 'short' }),
            temp: item.main.temp,
            description: item.weather[0].description,
            icon: item.weather[0].icon,
          })
        }
      })
      forecast3.value = days.slice(0, 3)
    }
  } catch (err) {
    console.error(err)
  }
}

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
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: system-ui, sans-serif;
}

.app {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background 0.5s ease;
  overflow-x: hidden;
}

.weather-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem 1.5rem;
  max-width: 100vw;
}

.weather-card {
  background: rgba(255, 255, 255, 0.2);
  padding: 2rem 3rem;
  border-radius: 2rem;
  backdrop-filter: blur(15px);
  text-align: center;
  color: #fff;
  min-width: 250px;
  max-width: 90vw;
  margin-bottom: 2rem;
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

.forecast {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 1rem;
  width: 100%;
  max-width: 100%;
}

.forecast-card {
  background: rgba(255, 255, 255, 0.15);
  padding: 1rem;
  border-radius: 1.5rem;
  backdrop-filter: blur(10px);
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  flex: 1 1 120px;
  max-width: 90vw;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.15);
  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.forecast-card:hover {
  transform: translateY(-5px) scale(1.03);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.25);
}

.forecast-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.forecast-enter-to {
  opacity: 1;
  transform: translateY(0);
}

.forecast-enter-active {
  transition: all 0.5s ease;
}

@media (max-width: 500px) {
  .weather-container {
    padding: 1rem 1rem;
  }

  .city-input {
    width: 100%;
  }

  .forecast-card {
    margin: 0.5rem 0;
    max-width: 100%;
  }
}
</style>
