<script setup>
import axios from 'axios'
import { ref, computed, onMounted } from 'vue'

const currentDate = ref(new Date())
const method = ref(15)
const latitudeAdjustmentMethod = ref(1)
const tune = ref('0,0,0,4,0,2,0,0,0')
const school = ref(1)
const prayerTimes = ref({})

const filteredPrayerTimes = computed(() =>
  Object.keys(prayerTimes.value).reduce((acc, key) => {
    if (['Fajr', 'Sunrise', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'].includes(key)) {
      acc[key] = prayerTimes.value[key]
    }
    return acc
  }, {})
)
const hijriDate = ref('')
const gregorianDate = ref('')

const getPosition = () => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject)
  })
}

const nextDay = () => {
  const newDay = new Date(currentDate.value)
  newDay.setDate(newDay.getDate() + 1)
  currentDate.value.setTime(newDay.getTime())
  fetchPrayerTimes(currentDate.value.toLocaleDateString('en-GB').replace(/\//g, '-'))
}

const previousDay = () => {
  const oldDay = new Date(currentDate.value)
  oldDay.setDate(oldDay.getDate() - 1)
  currentDate.value.setTime(oldDay.getTime())
  fetchPrayerTimes(currentDate.value.toLocaleDateString('en-GB').replace(/\//g, '-'))
}

const fetchPrayerTimes = async (date) => {
  const fdate = date.split('/').join('-')
  const position = await getPosition()
  const response = await axios.get(`https://api.aladhan.com/v1/timings/${fdate}`, {
    params: {
      latitude: position.coords.latitude,
      longitude: position.coords.longitude,
      method: method.value,
      latitudeAdjustmentMethod: latitudeAdjustmentMethod.value,
      tune: tune.value,
      school: school.value,
      adjustment: 1
    }
  })
  const data = response.data.data
  prayerTimes.value = data.timings
  const hijri = data.date.hijri
  const gregorian = data.date.gregorian
  hijriDate.value = `${hijri.month.en} ${hijri.day}, ${hijri.year} ${hijri.designation.abbreviated}`
  gregorianDate.value = `${gregorian.month.en} ${gregorian.day}, ${gregorian.year}`
}

onMounted(() => {
  fetchPrayerTimes(currentDate.value.toLocaleDateString())
  setInterval(() => {
    fetchPrayerTimes(currentDate.value.toLocaleDateString('en-GB').replace(/\//g, '-'))
  }, 3600000)
})
</script>

<template>
  <div>
    <h3>{{ gregorianDate }}</h3>
    <h3>{{ hijriDate }}</h3>
    <ul>
      <li v-for="(time, name) in filteredPrayerTimes" :key="name">{{ name }}: {{ time }}</li>
    </ul>
    <button @click="nextDay">next</button>
    <button @click="previousDay">before</button>
  </div>
</template>
