<template>
  <div v-for="city in savedCity" :key="savedCity.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import CityCard from "./CityCard.vue";
import axios from "axios";
import { useRouter } from "vue-router";
const openWeatherApi = import.meta.env.VITE_OPENWEATHER_KEY;
const savedCity = ref([]);
const router = useRouter();

const getCities = async () => {
  if (localStorage.getItem("savedCity")) {
    savedCity.value = JSON.parse(localStorage.getItem("savedCity"));
    const request = [];
    savedCity.value.forEach((city) => {
      request.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=${openWeatherApi}&units=metric`
        )
      );
    });

    const weatherData = await Promise.all(request);

    weatherData.forEach((values, index) => {
      savedCity.value[index].weather = values.data;
    });
  }
};
await getCities();

const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: { lat: city.coords.lat, lng: city.coords.lng },
  });
};
</script>
