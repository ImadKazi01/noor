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
  <div class="salaah">
    <div class="salaah__block">
      <span class="salaah__span">It is</span>
      <div class="salaah__name" v-if="currentPrayer">{{ currentPrayer }}</div>
    </div>
    <div class="salaah__next-block">
      <span class="salaah__next-span">Next salaah</span>
      <div class="salaah__next-name" v-if="nextPrayerTime">{{ nextPrayerTime }}</div>
    </div>
    <!--
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
    </p> -->
  </div>
</template>

<style lang="scss" scoped>
@import '../scss/global.scss';
.salaah {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  width: 100%;
  gap: 1rem;

  &__block {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
    border-radius: 1rem;
    width: 50%;
    background-color: $color-primary-100;
    gap: 1rem;
    padding: 1rem;
    box-shadow: 0 20px 15px -12px rgba(0, 0, 0, 0.1);
  }

  &__span {
    font-size: $font-size-xxs;
    font-weight: 400;
    color: $color-black;
  }

  &__name {
    font-size: $font-size-md;
    font-weight: 700;
    color: $color-primary;
  }

  &__next-block {
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: flex-start;
    border-radius: 1rem;
    width: 50%;
    background-color: $color-primary;
    gap: 1rem;
    padding: 1rem;
    box-shadow: 0 20px 15px -12px rgba(0, 0, 0, 0.1);
  }

  &__next-span {
    font-size: $font-size-xxs;
    font-weight: 400;
    color: $color-white;
  }

  &__next-name {
    font-size: $font-size-md;
    font-weight: 700;
    color: $color-secondary-100;
  }
}
</style>
