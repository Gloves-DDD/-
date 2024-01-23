<template>
  <div class="flex flex-col flex-1 items-center">
    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
      <p>你当前正在预览这个城市，单击“+”号 来开始追踪该城市的天气</p>
    </div>

    <!-- 今日天气预报内容 -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <P class="text-sm mb-12">最近观测时间 {{ weatherData.reporttime }}</P>
      <p class="text-8xl mb-8">{{ Math.round(weatherData.casts[0].daytemp) }}&deg;C</p>
      <p>
        今天的气温幅度在
        {{ Math.round(weatherData.casts[0].daytemp_float) }}&deg;C左右浮动
      </p>
      <p class="capitalize">
        {{ weatherData.casts[0].dayweather }}
      </p>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- 未来几天天气预报内容 -->
    <div class="max-w-screen-md w-full py-12">
      <div class="mx-8 text-white">
        <h2 class="mb-4">未来几天内的天气</h2>
        <div class="flex gap-10">
          <div
            v-for="castData in weatherData.casts"
            :key="castData.date"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">星期{{ castData.week }}</p>

            <p class="text-xl">
              H{{ Math.round(castData.daytemp) }}&deg; -- L{{ Math.round(castData.nighttemp) }}&deg;
            </p>
            <p class="text-[1rem]">{{ castData.dayweather }} - {{ castData.nightweather }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 移除已添加的城市天气信息 -->
    <div
      class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500"
      @click="removeCity"
    >
      <i class="fa-solid fa-trash"></i>
      <p>移除该城市天气信息</p>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'


// 获取天气信息
const route = useRoute()
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://restapi.amap.com/v3/weather/weatherInfo?key=705b37d7c01d515699b612a960cfb5f3&city=${route.query.adcode}&extensions=all`
    )

    await new Promise((res) => setTimeout(res, 1000))

    return weatherData.data.forecasts[0]
  } catch (err) {
    console.log(err)
  }
}
const weatherData = await getWeatherData()


// 移除已添加的城市天气信息
const router = useRouter()
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem('savedCities'))
  const updatedCities = cities.filter((city) => city.id !== route.query.id)
  localStorage.setItem('savedCities', JSON.stringify(updatedCities))
  router.push({
    name: 'home'
  })
}
</script>
