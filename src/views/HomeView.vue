<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
const searchQuery = ref("");
const queryTimeOut = ref(null);
const searchErrors = ref(null);
const apiKey = import.meta.env.VITE_API_KEY;
const mapBoxReults = ref(null);
const router = useRouter();

// lazy-search
const getSearchResults = () => {
  queryTimeOut.value = setTimeout(async () => {
    clearTimeout(queryTimeOut.value);
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${apiKey}&types=place`
        );
        mapBoxReults.value = result.data.features;
      } catch {
        searchErrors.value = true;
      }

      return;
    }
    mapBoxReults.value = null;
  }, 300);
};

const previewCity = (searchResults) => {
  console.log(searchResults, "LOG");
  const [city, state] = searchResults.place_name.split(",");
  router.push({
    name: "cityView",
    params: { state: state.replaceAll(" ", ""), city: city },
    query: {
      lat: searchResults.geometry.coordinates[1],
      lng: searchResults.geometry.coordinates[0],
      preview: true,
    },
  });
};
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        @input="getSearchResults"
        type="text"
        placeholder="Searching for the city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0#004e71]"
      />
      <ul
        class="absolue top-[66px] text-white bg-weather-secondary py-2 px-1 shadow-md"
        v-if="mapBoxReults"
      >
        <p v-if="searchErrors">Sorry , Something went wrong!!</p>
        <p v-if="!searchErrors && mapBoxReults.length === 0">
          No results match the query! Search Again
        </p>
        <template v-else>
          <li
            v-for="searchResults in mapBoxReults"
            :key="searchResults.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResults)"
          >
            {{ searchResults.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <p>Loading ...</p>
        </template>
      </Suspense>
    </div>
  </main>
</template>
