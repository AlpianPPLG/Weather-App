<template>
  <div
    :class="[
      'min-h-screen flex items-center justify-center',
      isDarkMode ? 'bg-gray-900' : 'bg-gray-100'
    ]"
  >
    <div
      :class="[
        'max-w-2xl w-full p-6 rounded-lg shadow-lg',
        isDarkMode ? 'bg-gray-800' : 'bg-white'
      ]"
    >
      <h1
        @click="reloadPage"
        :class="[
          'text-3xl font-bold mb-6 text-center cursor-pointer',
          isDarkMode ? 'text-white' : 'text-gray-800'
        ]"
      >
        Aplikasi Cuaca
      </h1>

      <!-- Tombol Dark Mode -->
      <button
        @click="toggleDarkMode"
        class="mb-4 px-4 py-2 rounded bg-indigo-500 text-white hover:bg-indigo-600"
      >
        {{ isDarkMode ? 'Light Mode' : 'Dark Mode' }}
      </button>

      <!-- Pilihan Bahasa -->
      <div class="mb-8">
        <label
          for="language"
          class="block text-sm font-medium"
          :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'"
          >Pilih Bahasa</label
        >
        <select
          v-model="selectedLanguage"
          @change="changeLanguage"
          id="language"
          class="mt-1 block w-full pl-3 pr-10 py-2 border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
        >
          <option value="id">Bahasa Indonesia</option>
          <option value="en">English</option>
        </select>
      </div>

      <!-- Pencarian Lokasi -->
      <div class="mb-8">
        <label
          for="location"
          class="block text-sm font-medium"
          :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'"
          >Cari Lokasi</label
        >
        <div class="mt-1 relative">
          <input
            v-model="searchQuery"
            @keyup.enter="validateAndSearch"
            type="text"
            id="location"
            name="location"
            :class="[
              'block w-full pl-3 pr-10 py-2 text-base border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm',
              isDarkMode ? 'bg-gray-700 text-white' : 'bg-white text-black'
            ]"
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
        <div class="p-6" :class="isDarkMode ? 'bg-gray-700' : 'bg-gray-200'">
          <div class="flex items-center justify-between">
            <div class="flex items-center space-x-4">
              <img
                :src="getWeatherIcon(weatherData.weather[0].icon)"
                alt="Weather Icon"
                class="w-16 h-16"
              />
              <div>
                <p :class="['text-xl font-semibold', isDarkMode ? 'text-white' : 'text-gray-800']">
                  {{ weatherData.name }}, {{ weatherData.sys.country }}
                </p>
                <p :class="['text-lg', isDarkMode ? 'text-gray-300' : 'text-gray-800']">
                  {{ weatherData.weather[0].description }}
                </p>
              </div>
            </div>
            <div class="flex items-center space-x-4">
              <p :class="['text-3xl font-semibold', isDarkMode ? 'text-white' : 'text-gray-800']">
                {{ weatherData.main.temp }}°C
              </p>
              <p :class="['text-gray-600', isDarkMode ? 'text-gray-400' : 'text-gray-600']">
                Feels like {{ weatherData.main.feels_like }}°C
              </p>
            </div>
          </div>
          <div class="mt-6 flex items-center space-x-8">
            <div>
              <p class="font-semibold" :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'">
                Humidity
              </p>
              <p>{{ weatherData.main.humidity }}%</p>
            </div>
            <div>
              <p class="font-semibold" :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'">
                Wind
              </p>
              <p>{{ weatherData.wind.speed }} m/s</p>
            </div>
            <div>
              <p class="font-semibold" :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'">
                Pressure
              </p>
              <p>{{ weatherData.main.pressure }} hPa</p>
            </div>
            <div>
              <p class="font-semibold" :class="isDarkMode ? 'text-gray-300' : 'text-gray-700'">
                Visibility
              </p>
              <p>{{ (weatherData.visibility / 1000).toFixed(1) }} km</p>
            </div>
          </div>
          <div class="mt-6">
            <p class="text-gray-700" :class="isDarkMode ? 'text-gray-400' : 'text-gray-700'">
              Updated at {{ formatDate(weatherData.dt * 1000) }}
            </p>
          </div>
        </div>

        <!-- Saran Aktivitas -->
        <div
          class="mt-6 p-4 border border-gray-300 rounded-md"
          :class="isDarkMode ? 'bg-gray-600' : 'bg-gray-100'"
        >
          <h2 :class="['text-lg font-bold', isDarkMode ? 'text-gray-200' : 'text-gray-800']">
            Saran Aktivitas
          </h2>
          <p :class="['mt-2', isDarkMode ? 'text-gray-300' : 'text-gray-700']">
            {{ getActivitySuggestion(weatherData.weather[0].main) }}
          </p>
        </div>
      </div>

      <!-- Prakiraan Cuaca Error -->
      <div v-else-if="weatherError" class="mt-8">
        <p class="text-red-500">{{ weatherError }}</p>
      </div>

      <!-- Pesan Kosong -->
      <div v-else class="mt-8">
        <p :class="['text-gray-600', isDarkMode ? 'text-gray-400' : 'text-gray-600']">
          Masukkan nama kota di Indonesia untuk melihat prakiraan cuaca.
        </p>
      </div>

      <!-- Berita Cuaca -->
      <div v-if="newsData" class="mt-8">
        <h2
          :class="[
            'text-2xl font-bold mb-4 text-center',
            isDarkMode ? 'text-gray-300' : 'text-gray-800'
          ]"
        >
          Berita Cuaca
        </h2>
        <ul>
          <li
            v-for="article in newsData.articles.slice(0, newsLimit)"
            :key="article.url"
            class="mb-4"
          >
            <a :href="article.url" target="_blank" class="text-blue-600 hover:underline">
              <h3
                :class="['text-lg font-semibold', isDarkMode ? 'text-gray-300' : 'text-gray-800']"
              >
                {{ article.title }}
              </h3>
            </a>
            <p :class="['text-sm', isDarkMode ? 'text-gray-400' : 'text-gray-600']">
              {{ article.description }}
            </p>
          </li>
        </ul>
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
      weatherError: null,
      newsData: null,
      isDarkMode: false,
      selectedLanguage: 'id', // Default bahasa
      newsLimit: 5 // Batas jumlah berita yang ditampilkan
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
      await this.searchWeather()
      await this.fetchWeatherNews()
    },
    async searchWeather() {
      try {
        const apiKey = '3d35e1cba82132db8f69ea0d65143b60' // Ganti dengan API key Anda
        const response = await fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.searchQuery},ID&appid=${apiKey}&units=metric&lang=${this.selectedLanguage}`
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
          text: error.message,
          confirmButtonColor: '#3085d6',
          confirmButtonText: 'OK'
        })
        this.weatherData = null
        this.weatherError = error.message
      }
    },
    async fetchWeatherNews() {
      try {
        const apiKey = '385287421d4e47a3bf6f7757044f3386' // Ganti dengan API key untuk berita
        const response = await fetch(
          `https://newsapi.org/v2/everything?q=weather&apiKey=${apiKey}&language=${this.selectedLanguage}`
        )
        if (!response.ok) {
          throw new Error('Berita tidak dapat dimuat!')
        }
        this.newsData = await response.json()
      } catch (error) {
        console.error(error.message)
        await Swal.fire({
          icon: 'error',
          title: 'Kesalahan!',
          text: 'Berita tidak dapat dimuat.',
          confirmButtonColor: '#3085d6',
          confirmButtonText: 'OK'
        })
        this.newsData = null
      }
    },
    getWeatherIcon(icon) {
      return `https://openweathermap.org/img/wn/${icon}.png`
    },
    formatDate(timestamp) {
      const date = new Date(timestamp)
      return date.toLocaleString(this.selectedLanguage === 'id' ? 'id-ID' : 'en-US', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        hour12: false
      })
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode
    },
    changeLanguage() {
      this.validateAndSearch() // Panggil pencarian ulang saat bahasa diubah
    },
    reloadPage() {
      window.location.reload()
    },
    getActivitySuggestion(weatherCondition) {
      switch (weatherCondition) {
        case 'Clear':
          return 'Ini adalah hari yang bagus untuk berjalan-jalan atau piknik di taman!'
        case 'Clouds':
          return 'Mungkin ini saat yang tepat untuk menjelajahi museum atau kafe.'
        case 'Rain':
          return 'Sebaiknya Anda tetap di dalam ruangan dan menikmati film atau membaca buku.'
        case 'Snow':
          return 'Waktu yang tepat untuk bermain salju atau membuat manusia salju!'
        case 'Thunderstorm':
          return 'Lebih baik tetap di dalam rumah dan menjauh dari jendela.'
        default:
          return 'Tidak ada saran aktivitas untuk kondisi cuaca ini.'
      }
    }
  }
}
</script>

<style>
canvas {
  max-width: 100%;
}
</style>
