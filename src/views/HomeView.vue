<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
          type="text"
          v-model="searchQuery"
          @input="getSearchResults"
          class="w-full py-2 px-1 bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
          placeholder="Search for city or state"
      >

      <ul
          v-if="mapboxSearchResult"
          class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p class="py-2" v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>


        <p
            class="py-2"
            v-if="!searchError && mapboxSearchResult.length === 0"
        >
          No results match your query, try a different term.
        </p>

        <template v-else>
          <li
              v-for="place in mapboxSearchResult"
              :key="place.id"
              class="py-2 cursor-pointer"
              @click="previewCity(place)"
          >{{ place.place_name }}</li>
        </template>
      </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CitesList />

        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import {ref} from "vue";
import axios from "axios";
import { useRouter } from 'vue-router'
import CitesList from "../components/CitesList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const mapboxAPIKey = 'pk.eyJ1IjoibWFrc3ltcmFrb21pbiIsImEiOiJjbGFxcjdnemMxOGVsM3ZucG12ZjZuOHRqIn0.twO0mhLEZPebAwsko0nAiQ'
const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResult = ref(null)
const searchError = ref(null)
const router = useRouter()

const getSearchResults = () => {
  queryTimeout.value = setTimeout(async () => {
    clearTimeout(queryTimeout.value)

    if (searchQuery.value !== '') {
      try {
        const result = await axios.get(
            `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        )

        mapboxSearchResult.value = result.data.features
      } catch (e) {
        searchError.value = true
      }

      return
    }

    mapboxSearchResult.value = null
  }, 300)
}

const previewCity = place => {
  console.log(place)
  const [city, state] = place.place_name.split(',')
  router.push({
    name: 'cityView',
    params: { state: state.trim(), city: city.trim() },
    query: {
      lat: place.geometry.coordinates[1],
      lng: place.geometry.coordinates[0],
      preview: true,
    },
  })
}

</script>
