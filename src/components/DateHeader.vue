<script setup>
import axios from 'axios'
import { ref, onMounted } from 'vue'

const currentDate = new Date()
const method = ref(15)
const latitudeAdjustmentMethod = ref(1)
const tune = ref('0,0,0,4,0,2,0,0,0')
const school = ref(1)

const hijriDate = ref('')
const gregorianDate = ref('')

const getPosition = () => {
  return new Promise((resolve, reject) => {
    navigator.geolocation.getCurrentPosition(resolve, reject)
  })
}

const fetchPrayerTimes = async () => {
  const position = await getPosition()
  const response = await axios.get(`https://api.aladhan.com/v1/timings/${currentDate.getDate}`, {
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
  const hijri = data.date.hijri
  const gregorian = data.date.gregorian
  hijriDate.value = `${hijri.month.en} ${hijri.day}, ${hijri.year} ${hijri.designation.abbreviated}`
  gregorianDate.value = `${gregorian.month.en} ${gregorian.day}, ${gregorian.year}`
}

onMounted(() => {
  fetchPrayerTimes()
  setInterval(() => {
    fetchPrayerTimes()
  }, 3600000)
})
</script>

<template>
  <div class="date">
    <div class="date__gregorian">{{ gregorianDate }}</div>
    <div class="date__hijri">{{ hijriDate }}</div>
  </div>
</template>

<style lang="scss" scoped>
@import '../scss/global.scss';
.date {
  &__gregorian {
    font-size: $font-size;
    font-weight: 700;
  }

  &__hijri {
    margin-top: 0.2rem;
    font-size: $font-size-xxs;
    font-weight: 600;
    color: $color-primary;
  }
}
</style>
