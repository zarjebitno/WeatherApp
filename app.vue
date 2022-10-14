<template>
  <div v-if="city" class="h-screen relative overflow-hidden">
    <img :src="background" alt=""/>
    <div class="absolute w-full h-full top-0 overlay"/>
    <div class="absolute w-full h-full top-0 p-48">
      <div class="flex justify-between">
        <div>
        <h1 class="text-7xl text-white">{{ city.name }}</h1>
        <p class="font-extralight text-2xl mt-2 text-white">{{ getDateTime(city.timezone) }}</p>
        <img :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@4x.png`" alt="" class="w-256 icon"/>
      </div>
      <div>
        <p class="text-9xl text-white font-extralight">{{ kelvinToCelsius(city.main.temp) }} °C</p>
      </div>
      </div>
      <div class="mt-20">
        <input type="text" class="w-1/2 h-10" placeholder="Search a city" v-model="input"/>
        <button class="bg-sky-400 w-20 text-white h-10" @click="handleSubmit">Search</button>
      </div>
    </div>
  </div>
  <div v-else class="p-10">
    <h1 class="text-7xl">Cannot find that city</h1>
    <button class="mt-5 bg-sky-400 px-10 w-50 text-white h-10" @click="goBack">Go Back</button>
  </div>
</template>

<script setup lang="ts">
  // composables
  const cookie = useCookie('city')
  const config = useRuntimeConfig()

  if(!cookie.value) {
    cookie.value = "Belgrade"
  }

  const search = ref(cookie.value)
  const input = ref("")
  const background = ref("")

  const {data: city, error} = useAsyncData(
    'city', 
    async () => {
      let response;
      
      try {
        response = await $fetch(
          `https://api.openweathermap.org/data/2.5/weather?q=${search.value}`,
          {
            params: {
              appId: config.WEATHER_APP_SECRET
            }
          }
        )

        cookie.value = search.value

        const temp = response.main.temp;

        if (temp <= 0) {
          background.value =
            "https://images.unsplash.com/photo-1483664852095-d6cc6870702d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
        } else if (temp > 0 && temp <= 10) {
          background.value =
            "https://images.unsplash.com/photo-1476820865390-c52aeebb9891?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3540&q=80";
        } else if (temp > 10 && temp <= 20) {
          background.value =
            "https://images.unsplash.com/photo-1560258018-c7db7645254e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=4032&q=80";
        } else {
          background.value =
            "https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=3546&q=80";
        }

        return response

      } catch(e) {
        console.log(e)
      }

      return response
    }, {
      watch: [search]
    })

  const handleSubmit = () => {
    const formatedSearch = input.value.trim().split(" ").join("+")
    search.value = formatedSearch
    input.value = ""
    console.log(city)
  }

  const goBack = () => {
    search.value = cookie.value
  }

  const kelvinToCelsius = (kelvin) => {
    return Math.round(kelvin - 273.15)
  }

  const getDateTime = (secondsDifferential) => {
    const currentDate = new Date()
    const timezoneOffset = currentDate.getTimezoneOffset() * 60 * 1000
    const zeroTime = currentDate.getTime() + timezoneOffset
    const currentLocalTimestamp = zeroTime + (secondsDifferential * 1000)

    const localDateTime = new Date(currentLocalTimestamp)
    const formatedDate = localDateTime.toLocaleString()

    return formatedDate

  }

</script>

<style scoped>
  .overlay {
    background: rgba(0, 0, 0, 0.5);
  }
  .icon {
    margin-left: -2.75rem;
    margin-top: -2.75rem;
  }
</style>
