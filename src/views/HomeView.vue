<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapSearchResults"
      >
        <P v-if="searchError"> 抱歉，似乎出了什么问题 TAT </P>
        <p v-if="!serverError && mapSearchResults.length === 0">你的查询并无任何匹配结果</p>
        <template v-else>
          <li
            v-for="searchResult in mapSearchResults"
            :key="searchResult.id"
            class="py-1 cursor-pointer text-[1rem]"
            @click="previewCity(searchResult)"
          >
            {{ searchResult.pname }} , {{ searchResult.adname }} , {{ searchResult.name }}
          </li>
        </template>
      </ul>
    </div>
    
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '@/components/CityList.vue';
import CityCardSkeleton from '@/components/CityCardSkeleton.vue';

const router = useRouter()

const previewCity = (searchResult) => {
  const state = searchResult.adname
  const city = searchResult.cityname
  const adcode = searchResult.adcode
  router.push({
    name: 'cityView',
    params: { state: state, city: city },
    query: {
      adcode: adcode,
      preview: true,
    },
  })
}

const searchQuery = ref('')
const queryTimeout = ref(null)
const mapSearchResults = ref(null)
const searchError = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
          `https://restapi.amap.com/v3/place/text?key=705b37d7c01d515699b612a960cfb5f3&keywords=${searchQuery.value}&offset=10&extensions=all`
        )
        mapSearchResults.value = result.data.pois
      } catch {
        searchError.value = true
      }
      return
    }
    mapSearchResults.value = null
  }, 300)
}
</script>
