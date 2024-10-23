<script setup>

import { ref, onMounted } from 'vue'

const wines = ref([]);
const modalOpen = ref(false);
const selectedWine = ref(null);

const getTotalRatings = (wines) => {
  return wines.reduce((total, wine) => {
        let rating = parseInt(wine.rating.reviews.split(' ')[0]);
        return total + (isNaN(rating) ? 0 : rating);
      }, 0)
}

const getMostReprecentedCountry = (wines) => {
  let countries = {};
  wines.forEach(wine => {
    let country = wine.location.split('\n·\n')[0];
    countries[country] = countries[country] ? countries[country]++ : 1;
  })
  let countryWithHighestCount = Object.entries(countries).sort((a, b) => b[1] - a[1])[0];
  return countryWithHighestCount?.[0]
}

const getName = (wine) => {
 return wine.wine.split(' ').slice(0, -1).join(' ')
}

const getLocation = (wine) => {
 return wine.location.split('\n·\n')[0]
}

const getYear = (wine) => {
 return wine.wine.split(' ').slice(-1)[0]
}

const setModalOpen = (wine) => {
  selectedWine.value = wine;
  modalOpen.value = true;
} 

const setModalClosed = () => {
  modalOpen.value = false;
  selectedWine.value = null;
} 

const fetchData = async () => {
  try {
    const res = await fetch('https://api.sampleapis.com/wines/whites');
    const data = await res.json();
    // Just taking the 5 first wines as I didn't implement pagination for this
    wines.value = data.slice(0,5);
  } catch (error) {
    console.error('Error fetching wines:', error);
  }
}

onMounted(fetchData)
</script>

<template>
  <div class='w-full h-screen bg-white relative overflow-hidden sm:pb-14 sm:pt-4 sm:w-4/5 sm:h-full sm:rounded-2xl sm:m-auto sm:mt-12 sm:shadow-[0px_2px_4px_-3px_#00000040]'>
    <div class='px-4 pt-8 bg-[#F7F7FA] sm:bg-white sm:px-12'>
      <h1 class='text-[42px] font-bold mb-7 sm:text-[32px]'>Wines</h1>
      <div class='flex flex-wrap md:gap-20'>
        <div class='flex-[0_0_100%] mb-7 md:flex-[unset]'>
          <p class='text-[#59677D] text-xs'>Total rated wines</p>
          <p class='text-[32px] font-bold sm:text-[20px]'>{{`${wines.length} wines`}}</p>
        </div>
        <div class='flex-[0_0_50%] mb-7 md:flex-[unset]'>
          <p class='text-[#59677D] text-xs'>Total ratings</p>
          <p class='text-[24px] font-bold sm:text-[20px]'>{{`${getTotalRatings(wines)} raitings`}}</p>
        </div>
        <div class='flex-[0_0_50%] mb-7 md:flex-[unset]'>
          <p class='text-[#59677D] text-xs'>Most represented country</p>
          <p class='text-[24px] font-bold sm:text-[20px]'>{{getMostReprecentedCountry(wines)}}</p>
        </div>
      </div>
    </div>
    <table class='w-screen text-left sm:w-full'>
      <thead>
        <tr class='text-[#59677D] text-xs h-12 border-b-[1px] border-[#59677D1A] sm:h-9'>
          <th class='text-right'>Rating</th>
          <th>Title</th>
          <th class='hidden sm:table-cell'>Country</th>
          <th>Year</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="wine in wines" :key="wine.id" class='text-xs h-12 p-2 sm:h-9' @click="setModalOpen(wine)">
            <td class='font-bold text-right'>{{wine.rating.average}}</td>
            <td>{{getName(wine)}}</td>
            <td class='hidden sm:table-cell'>{{getLocation(wine)}}</td>
            <td class='w-1/4 sm:w-1/6'>{{getYear(wine)}}</td>
        </tr>
      </tbody>
    </table>
    <button class="text-[#0066FF] text-base mx-auto w-full mt-8 sm:text-sm sm:mt-10">Show more</button>
    <div v-if="modalOpen" class="w-screen h-screen fixed top-0 left-0 sm:bg-[#59677D8A]">
      <div class="w-full h-full bg-white px-4 py-20 sm:w-[580px] sm:h-auto sm:rounded-2xl sm:relative sm:m-auto sm:mt-24">
        <button @click="setModalClosed" class="absolute right-4 top-4">X</button>
        <p class='mb-7'>{{`Details of the wine "${selectedWine.wine}" goes here.`}}</p>
        <button class="text-[#0066FF] text-base mx-auto w-full mt-8 sm:text-sm" @click="setModalClosed">Close</button>
      </div>
    </div>
  </div>
  
</template>

<style scoped>
  tbody tr:hover,
  tbody tr:active,
  tbody tr:focus {
    color:#0066FF;
    background-color: #F7F7FA;
    border-radius: 8px;
  }
  th {
  padding: 16px;
  font-weight: 400;
  }
  td {
  padding: 12px 16px 12px 16px;
  }
</style>
