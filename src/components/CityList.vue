<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)"/>
  </div>

  <p v-if="savedCities.length === 0">没有被追踪的城市，先去添加一个城市看看吧。</p>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';

// 填充获取的城市列表
const savedCities = ref([])
const getCities = async () => {
  // 检测本地仓库：savedcities是否存在
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'))
    // 充当中间数组
    const requests = []
    // 填充城市列表数组
    savedCities.value.forEach((city) => {
      requests.push(
        axios.get(`https://restapi.amap.com/v3/weather/weatherInfo?key=705b37d7c01d515699b612a960cfb5f3&city=${city.adcode}&extensions=all`)
      )
    })

    const weatherData = await Promise.all(requests)
    
    // 人为设定延迟为1000ms
    await new Promise((res) => setTimeout(res, 1000))
    
    weatherData.forEach((value, index) => {
      savedCities.value[index].weather = value.data
    })
  }
}
await getCities()

const router = useRouter()
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params:{ state: city.state, city: city.city},
    query: {
      id: city.id,
      adcode: city.adcode,
    }
  })
}
</script>