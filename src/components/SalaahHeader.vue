<script setup>
import axios from 'axios'
import { ref, computed, onMounted, watch } from 'vue'

const method = ref(15)
const latitudeAdjustmentMethod = ref(1)
const tune = ref('0,0,0,4,0,2,0,0,0')
const school = ref(1)
const currentDate = new Date()

const prayerTimes = ref({})
const filteredPrayerTimes = computed(() =>
  Object.keys(prayerTimes.value).reduce((acc, key) => {
    if (['Fajr', 'Sunrise', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'].includes(key)) {
      acc[key] = prayerTimes.value[key]
    }
    return acc
  }, {})
)

const currentPrayer = ref('')
const nextPrayerTime = ref('')
const nextPrayerTimeValue = ref('')
const timeDifference = ref('')

const fetchPrayerTimes = async () => {
  navigator.geolocation.getCurrentPosition(
    async (position) => {
      const response = await axios.get(
        `https://api.aladhan.com/v1/timings/${currentDate.getDate}`,
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
      determineCurrentPrayer() // Call the function here
    },
    (error) => {
      console.error(error)
    }
  )
}

const determineCurrentPrayer = () => {
  const currentTime = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
  const prayerTimesValues = Object.values(prayerTimes.value)
  const prayerNames = Object.keys(filteredPrayerTimes.value)
  const index = prayerTimesValues.findIndex((time) => currentTime <= time)
  const nextPrayerIndex = index === -1 || index === prayerTimesValues.length - 1 ? 0 : index
  currentPrayer.value = index === 0 || index === -1 ? '' : prayerNames[index - 1]
  nextPrayerTime.value = prayerNames[nextPrayerIndex]

  // Calculate the time until the next prayer
  nextPrayerTimeValue.value = new Date().setHours(
    prayerTimesValues[nextPrayerIndex].split(':')[0],
    prayerTimesValues[nextPrayerIndex].split(':')[1],
    0
  )
  timeDifference.value = nextPrayerTimeValue.value - new Date()
}

watch(prayerTimes, () => determineCurrentPrayer())

onMounted(() => {
  fetchPrayerTimes()
  setInterval(() => {
    fetchPrayerTimes()
  }, 3600000)
})
setInterval(() => determineCurrentPrayer(), 1000)
</script>

<template>
  <div class="header">
    <div class="header__top">
      <h1 class="header__salaah" v-if="currentPrayer">{{ currentPrayer }}</h1>
      <p class="header__next" v-if="currentPrayer && timeDifference > 0">
        {{ nextPrayerTime }} is at
        {{
          new Date(nextPrayerTimeValue).toLocaleTimeString([], {
            hour: '2-digit',
            minute: '2-digit'
          })
        }}
      </p>
    </div>
    <p v-if="currentPrayer && timeDifference > 0">
      {{
        Math.floor(timeDifference / (1000 * 60 * 60)) > 0
          ? Math.floor(timeDifference / (1000 * 60 * 60)) + ' hours, '
          : ''
      }}
      {{
        Math.floor((timeDifference / (1000 * 60)) % 60) > 0
          ? Math.floor((timeDifference / (1000 * 60)) % 60) + ' minutes'
          : ''
      }}
      until {{ nextPrayerTime }}
    </p>
  </div>
</template>

<style lang="scss" scoped>
@import '../scss/global.scss';
.header {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: center;
  height: 100%;
  width: 100%;
  color: var(--color-text);
  gap: 1rem;

  &__top {
    display: flex;
    flex-direction: row;
    align-items: flex-start;
    justify-content: center;
    width: 100%;
    gap: 1rem;
  }

  &__salaah {
    font-size: 2rem;
    font-weight: 600;
  }

  &__salaah,
  &__next {
    width: 50%;
    margin: 0;
    padding: 4rem 3rem;
    background-color: $color-white;
    border-radius: 1.5rem;
    box-shadow: 2.2px 3.6px 4px rgba(0, 0, 0, 0.022), 5.6px 9.2px 10.2px rgba(0, 0, 0, 0.031),
      11.5px 18.8px 20.7px rgba(0, 0, 0, 0.039), 23.7px 38.7px 42.7px rgba(0, 0, 0, 0.048),
      65px 106px 117px rgba(0, 0, 0, 0.07);
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>
