<script setup>
import Header from '../components/SalaahHeader.vue';
import DayTimes from '../components/DayTimes.vue';
import DateHeader from '../components/DateHeader.vue';
</script>

<template>
  <div class="noor">
    <DateHeader />
    <Header />
    <!-- <DayTimes /> -->
  </div>
</template>

<style lang="scss" scoped>
@import '../scss/global.scss';

.noor {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}
</style>