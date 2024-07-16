<template>
  <div class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="max-w-2xl w-full p-6 bg-white rounded-lg shadow-lg">
      <h1
        @click="reloadPage"
        class="text-3xl font-bold mb-6 text-center text-gray-800 cursor-pointer"
      >
        Aplikasi Cuaca
      </h1>

      <!-- Pencarian Lokasi -->
      <div class="mb-8">
        <label for="location" class="block text-sm font-medium text-gray-700">Cari Lokasi</label>
        <div class="mt-1 relative">
          <input
            v-model="searchQuery"
            @keyup.enter="searchWeather"
            type="text"
            id="location"
            name="location"
            class="block w-full pl-3 pr-10 py-2 text-base border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            placeholder="Masukkan nama kota di Indonesia..."
          />
          <button
            @click="validateAndSearch"
            type="button"
            class="absolute right-0 top-0 mt-2 mr-3 px-4 py-2 bg-indigo-500 text-white rounded-md shadow-sm hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            Cari
          </button>
        </div>
      </div>

      <!-- Prakiraan Cuaca -->
      <div v-if="weatherData" class="mt-8">
        <div class="p-6 bg-gray-200 rounded-lg shadow-md">
          <div class="flex items-center justify-between">
            <div class="flex items-center space-x-4">
              <img
                :src="getWeatherIcon(weatherData.weather[0].icon)"
                alt="Weather Icon"
                class="w-16 h-16"
              />
              <div>
                <p class="text-xl font-semibold">
                  {{ weatherData.name }}, {{ weatherData.sys.country }}
                </p>
                <p class="text-lg text-gray-800">{{ weatherData.weather[0].description }}</p>
              </div>
            </div>
            <div class="flex items-center space-x-4">
              <p class="text-3xl font-semibold">{{ weatherData.main.temp }}°C</p>
              <p class="text-gray-600">Feels like {{ weatherData.main.feels_like }}°C</p>
            </div>
          </div>
          <div class="mt-6 flex items-center space-x-8">
            <div>
              <p class="font-semibold text-gray-700">Humidity</p>
              <p>{{ weatherData.main.humidity }}%</p>
            </div>
            <div>
              <p class="font-semibold text-gray-700">Wind</p>
              <p>{{ weatherData.wind.speed }} m/s</p>
            </div>
            <div>
              <p class="font-semibold text-gray-700">Pressure</p>
              <p>{{ weatherData.main.pressure }} hPa</p>
            </div>
            <div>
              <p class="font-semibold text-gray-700">Visibility</p>
              <p>{{ (weatherData.visibility / 1000).toFixed(1) }} km</p>
            </div>
          </div>
          <div class="mt-6">
            <p class="text-gray-700">Updated at {{ formatDate(weatherData.dt * 1000) }}</p>
          </div>
        </div>
      </div>
      <div v-else-if="weatherError" class="mt-8">
        <p class="text-red-500">{{ weatherError }}</p>
      </div>
      <div v-else class="mt-8">
        <p class="text-gray-600">Masukkan nama kota di Indonesia untuk melihat prakiraan cuaca.</p>
      </div>
    </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'

export default {
  name: 'App',
  data() {
    return {
      searchQuery: '',
      weatherData: null,
      weatherError: null
    }
  },
  methods: {
    async validateAndSearch() {
      if (!this.searchQuery) {
        await Swal.fire({
          icon: 'error',
          title: 'Kesalahan!',
          text: 'Kolom pencarian tidak boleh kosong.',
          confirmButtonColor: '#3085d6',
          confirmButtonText: 'OK'
        })
        return
      }
      this.searchWeather()
    },
    async searchWeather() {
      try {
        const apiKey = '3d35e1cba82132db8f69ea0d65143b60' // Ganti dengan API key Anda
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.searchQuery},ID&appid=${apiKey}&units=metric`
        )
        if (!response.ok) {
          throw new Error('Kota tidak ditemukan!')
        }
        this.weatherData = await response.json()
        this.weatherError = null
      } catch (error) {
        console.error(error.message)
        await Swal.fire({
          icon: 'error',
          title: 'Kesalahan!',
          text: 'Kota tidak ditemukan.',
          confirmButtonColor: '#3085d6',
          confirmButtonText: 'OK'
        })
        this.weatherData = null
        this.weatherError = error.message
      }
    },
    getWeatherIcon(icon) {
      return `https://openweathermap.org/img/wn/${icon}.png`
    },
    formatDate(timestamp) {
      const date = new Date(timestamp)
      return date.toLocaleString('id-ID', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        hour12: false
      })
    },
    reloadPage() {
      window.location.reload()
    }
  }
}
</script>
