<template>
    <main class="container text-white">
        <div class="pt-4 mb-8 relative">
            <input type="text"
                   v-model="searchQuery"
                   @input="getSearchResult"
                   placeholder="Search for the city or state"
                   class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:shadow-[0px_1px_0_0_#004E71] focus:outline-none"
            >
            <ul v-if="mapboxResult" class="absolute rounded-xl bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
                <p v-if="searchError">
                    Sorry, something went wrong, please try again.
                </p>
                <p class="p-2" v-if="!searchError && mapboxResult?.length === 0">
                    No results match your query, try a different term.
                </p>
                <template v-else>
                    <li v-for="item in mapboxResult"
                        :key="item.id"
                        class="p-2 cursor-pointer"
                        @click="previewCity(item)"
                    >
                        {{item.place_name}}
                    </li>
                </template>
            </ul>
        </div>
        <div class="flex flex-col gap-4">
            <Suspense>
                <CityList/>
                <template #fallback>
                    <div class="flex justify-center mt-5">
                        <div class="lds-dual-ring"></div>
                    </div>
                </template>
            </Suspense>
        </div>
    </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "@/components/CityList.vue";

const router = useRouter()
const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxResult = ref(null)
const searchError = ref(null)
const mapboxApiKey = 'pk.eyJ1IjoiZGlhbmFzaGVwdGFrIiwiYSI6ImNsaTB0aWwxMzAwbGUzamxoN3BrMW9lanoifQ.oyHLjPwxnmMClkMN81QqCw'
const getSearchResult = () => {
    clearTimeout(queryTimeout.value)
    queryTimeout.value = setTimeout(async () => {
        if (searchQuery.value !== '') {
            try {
                const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxApiKey}`)
                mapboxResult.value = result.data.features;
                console.log(mapboxResult.value)
                return;
            } catch {
                searchError.value = true
            }
        }
        mapboxResult.value = null
    }, 300)
}

const previewCity = (searchResult) => {
    const [city, state] = searchResult.place_name.split(",")
    router.push({
        name: "cityView",
        params: {
            state: state.trim(),
            city: city
        },
        query: {
            lat: searchResult.geometry.coordinates[1],
            lng: searchResult.geometry.coordinates[0],
            preview: true
        }
    })

}

</script>

<style scoped>
.lds-dual-ring {
    display: inline-block;
    width: 80px;
    height: 80px;
}
.lds-dual-ring:after {
    content: " ";
    display: block;
    width: 64px;
    height: 64px;
    margin: 8px;
    border-radius: 50%;
    border: 6px solid #fff;
    border-color: #fff transparent #fff transparent;
    animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

</style>
