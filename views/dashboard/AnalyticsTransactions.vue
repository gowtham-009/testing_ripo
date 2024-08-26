<script setup lang="ts">
import { ref } from 'vue';

const Week1 = [
  {
    title: 'Sales',
    stats: '245k',
    icon: 'ri-pie-chart-2-line',
    color: 'primary',
  },
  {
    title: 'Customers',
    stats: '12.5k',
    icon: 'ri-group-line',
    color: 'success',
  },
  {
    title: 'Product',
    stats: '1.54k',
    icon: 'ri-macbook-line',
    color: 'warning',
  },
  {
    title: 'Revenue',
    stats: '$88k',
    icon: 'ri-money-dollar-circle-line',
    color: 'info',
  },
  {
    title: 'E-5',
    stats: '$98k',
    icon: 'ri-money-dollar-circle-line',
    color: 'dark',
  },
  {
    title: 'E-6',
    stats: '$27',
    icon: 'ri-money-dollar-circle-line',
    color: 'warning',
  },
  {
    title: 'E-7',
    stats: '$58k',
    icon: 'ri-money-dollar-circle-line',
    color: 'secondary',
  },
  {
    title: 'E-8',
    stats: '$87k',
    icon: 'ri-money-dollar-circle-line',
    color: 'primary',
  },
]

const week2 = [
  {
    title: 'Sales2',
    stats: '7.26k',
    icon: 'ri-pie-chart-2-line',
    color: 'primary',
  },
  {
    title: 'Customers2',
    stats: '8.6k',
    icon: 'ri-group-line',
    color: 'success',
  },
  {
    title: 'Product2',
    stats: '7.98k',
    icon: 'ri-macbook-line',
    color: 'warning',
  },
  {
    title: 'Revenue2',
    stats: '$23.5k',
    icon: 'ri-money-dollar-circle-line',
    color: 'info',
  },
  {
    title: 'E-52',
    stats: '$68k',
    icon: 'ri-money-dollar-circle-line',
    color: 'dark',
  },
  {
    title: 'E-62',
    stats: '$25',
    icon: 'ri-money-dollar-circle-line',
    color: 'warning',
  },
  {
    title: 'E-72',
    stats: '$98k',
    icon: 'ri-money-dollar-circle-line',
    color: 'secondary',
  },
  {
    title: 'E-82',
    stats: '$87k',
    icon: 'ri-money-dollar-circle-line',
    color: 'primary',
  },
]

const selectedWeek = ref('Week1')

const currentData = computed(() => {
  return selectedWeek.value === 'week2' ? week2 : Week1
})
const items = ref<string[]>(['week1', 'week2']);
</script>

<template>
  <VCard title="Transactions">
    <template #subtitle>
      <p class="text-body-1 mb-0">
        <span class="d-inline-block font-weight-medium text-high-emphasis">Total 48.5% Growth</span> <span class="text-high-emphasis">😎</span> this month
      </p>
    </template>

    <template #append> 
      <v-select
      :items="items"
     
      v-model="selectedWeek"
    ></v-select>
    </template>

    <VCardText class="pt-10">
      <VRow>
        <VCol
          v-for="item in currentData"
          :key="item.title"
          cols="12"
          sm="6"
          md="3"
        >
          <div class="d-flex align-center gap-x-3">
            <VAvatar
              :color="item.color"
              rounded
              size="40"
              class="elevation-2"
            >
              <VIcon
                size="24"
                :icon="item.icon"
                class=""
              />
            </VAvatar>
            <div class="d-flex flex-column">
              <div class="text-body-1">
                {{ item.title }}
              </div>
              <h5 class="text-h5">
                {{ item.stats }}
              </h5>
            </div>
          </div>
        </VCol>
      </VRow>
    </VCardText>
  </VCard>
</template>
