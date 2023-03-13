<script setup>
import axios from 'axios'
import { ref, computed, onMounted, watch } from 'vue'

const currentDate = new Date()
const formattedCurrentDate = `${currentDate.getDate()}-${
  currentDate.getMonth() + 1
}-${currentDate.getFullYear()}`

const method = ref(15)
const latitudeAdjustmentMethod = ref(1)
const tune = ref('0,0,0,4,0,2,0,0,0')
const school = ref(1)

const formattedDate = computed(() => {
  return formattedCurrentDate
})

const prayerTimes = ref({})
const hijriDate = ref({})
const gDate = ref({})

const fetchPrayerTimes = async () => {
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const response = await axios.get(
        `https://api.aladhan.com/v1/timings/${formattedCurrentDate}`,
        {
          params: {
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
            method: method.value,
            latitudeAdjustmentMethod: latitudeAdjustmentMethod.value,
            tune: tune.value,
            school: school.value
          }
        }
      )
      prayerTimes.value = response.data.data.timings
      hijriDate.value = response.data.data.date.hijri
      gDate.value = response.data.data.date.gregorian
    },
    (error) => {
      console.error(error)
    }
  )
}

console.log(hijriDate)

const filteredPrayerTimes = computed(() => {
  const { Fajr, Sunrise, Dhuhr, Asr, Maghrib, Isha } = prayerTimes.value
  return { Fajr, Sunrise, Dhuhr, Asr, Maghrib, Isha }
})

const currentPrayer = ref('')
const nextPrayerTime = ref('')

const determineCurrentPrayer = () => {
  const currentTime = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
  const prayerTimesValues = Object.values(prayerTimes.value)
  const prayerNames = Object.keys(filteredPrayerTimes.value)
  const index = prayerTimesValues.findIndex((time) => currentTime <= time)

  if (index === 0) {
    currentPrayer.value = ''
    nextPrayerTime.value = ''
  } else if (index === -1) {
    currentPrayer.value = ''
    nextPrayerTime.value = ''
  } else {
    currentPrayer.value = prayerNames[index - 1]
    nextPrayerTime.value = prayerNames[index]
  }
}

watch(prayerTimes, () => {
  determineCurrentPrayer()
})

onMounted(() => {
  fetchPrayerTimes()
})
</script>

<template>
  <div>
    <h1 v-if="currentPrayer">{{ currentPrayer }}</h1>
    <p>{{ nextPrayerTime }} is in</p>
    <h3>Prayer Times for {{ formattedDate }}</h3>
    <!-- <h3>{{ hijriDate.month.en }} {{ hijriDate.day }}, {{ hijriDate.year }} {{ hijriDate.designation.abbreviated }}</h3> -->
    <ul>
      <li v-for="(time, name) in filteredPrayerTimes" :key="name">{{ name }}: {{ time }}</li>
    </ul>
  </div>
</template>
