<template>
  <main class="container text-white">
      <div class="pt-4 mb-8 relative">
          <input type="text" v-model="searchQuery" @input="getSearchResults" placeholder="Search for a city or state" class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
          <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]" v-if="weatherSearchResults">
            <p v-if="searchError">Sorry, something went wrong, please try again.</p>
            <p v-if="!searchError && weatherSearchResults.length === 0">No results match your query, try a different term.</p>
            <template v-else>
              <li v-for="searchResult in weatherSearchResults" :key="searchResult.id" class="py-2 cursor-pointer" @click="previewCity(searchResult)">
                {{ searchResult.country }}, {{ searchResult.name }}
              </li>
            </template>
          </ul>
      </div>
      <div class="flex flex-col gap-4">
        <Suspense>
          <CityList/>
          <template #fallback>
            <CityCardSkeleton />
          </template>
        </Suspense>
      </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const city= searchResult.name;
  const state = searchResult.country;
  router.push({
    name: 'cityView',
    params: { state: state.replaceAll(" ", ""), city: city}, 
    query: {
      lat: searchResult.lat,
      lng: searchResult.lon,
      preview: true,
    }
  });
};

const weatherAPIKey = "6513c90750ea4474945171638232309";
const searchQuery = ref("");
const queryTimeout = ref(null);
const weatherSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
      if (searchQuery.value !== "") {
        try {
          const result = await axios.get(`https://api.weatherapi.com/v1/search.json?key=${weatherAPIKey}&q=${searchQuery.value}`);
          weatherSearchResults.value = result.data;
          // console.log(weatherSearchResults.value);
        }
        catch {
          searchError.value = true;
        }
          return;
      };
      weatherSearchResults.value = null;
  }, 300);
};
// https://api.weatherapi.com/v1/current.json?key=6513c90750ea4474945171638232309&q=London&aqi=no
// http://api.weatherapi.com/v1/search.json?key=6513c90750ea4474945171638232309&city=Lo
// /search.json 
</script>