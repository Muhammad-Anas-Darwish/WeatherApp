<template>
    <div class="flex flex-col flex-1 items-center">
        <!-- Banner -->
        <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
            <p>You are currently previewing this city, click the "+" icon to start tracking this city.</p>
        </div>
        <!-- Weather Overview -->
        <div class="flex flex-col items-center text-white py-12">
            <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
            <p class="text-sm mb-12">
                {{ 
                    new Date(weatherData.location.localtime).toLocaleDateString(
                        "en-us", 
                        {
                            weekday: "short",
                            day: "2-digit",
                            month: "long",
                        }
                    ) 
                }}
                {{ 
                    new Date(weatherData.location.localtime).toLocaleTimeString(
                        "en-us",
                        {
                            timeStyle: "short",
                        }
                    )
                }}
            </p>
            <p class="text-8xl mb-8">{{ Math.round(weatherData.current.temp_c) }}&deg</p>
            <p>
                Feels like
                {{ Math.round(weatherData.current.feelslike_c) }}&deg
            </p>
            <p class="capitalize">
                {{ weatherData.current.condition.text }}
            </p>
            <img class="w-[150px] h-auto" :src="weatherData.current.condition.icon" alt="">
        </div>

        <hr class="border-white border-opacity-10 border w-full">

        <!-- Hourly Weather -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">Hourly Weather</h2>
                <div class="flex gap-10 overflow-x-scroll">
                    <div v-for="hourData in weatherData.forecast.forecastday[0].hour" :key="weatherData.time" class="flex flex-col gap-4 items-center">
                        <p class="whitespace-nowrap text-md">
                            {{
                                new Date(
                                    hourData.time
                                    ).toLocaleTimeString("en-us", {
                                        hour:"numeric",
                                })
                            }}
                        </p>
                        <img class="w-auto h-[50px] object-cover" :src="hourData.condition.icon" alt="">
                        <p class="text-xl">
                            {{ Math.round(hourData.temp_c) }}&deg
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <hr class="border-white border-opacity-10 border w-full">

        <!-- Weekly Weather -->
        <div class="max-w-screen-md w-full py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4">7 Day Forecast</h2>
                <div v-for="day in weatherData.forecast.forecastday" :key="day.date" class="flex items-center">
                    <p class="flex-1">
                        {{ 
                            new Date(day.date).toLocaleDateString(
                                "en-us", 
                                {
                                    weekday: "long",
                                }
                            ) 
                        }}
                    </p>
                    <img class="w-[50px] h-[50px] object-cover" :src="day.day.condition.icon" alt="">
                    <div class="flex gap-2 flex-1 justify-end">
                        <p>H: {{ Math.round(day.day.maxtemp_c) }}</p>
                        <p>L: {{ Math.round(day.day.mintemp_c) }}</p>
                    </div>
                </div>
            </div>
        </div>

        <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity">
            <i class="fa-solid fa-trash"></i>
            <p>Remove City</p>
        </div>
    </div>
</template>

<script setup>
import axios from 'axios';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const getWeatherData = async () => {
    try {
        const weatherData = await axios.get(
            `https://api.weatherapi.com/v1/forecast.json?key=6513c90750ea4474945171638232309&days=7&q=${route.query.lat},${route.query.lng}`
        );
        console.log(weatherData);
        // // cal current date & time
        // const localOffset = new Date().getTimezoneOffset() * 60000;
        // const utc = weatherData.data.current.dt * 1000 + localOffset;
        // weatherData.data.currentTime = utc + 1000 * weatherData.data.timezone_offset;

        // // cal hourly weather offset
        // weatherData.data.hourly.forEach((hour) => {
        //     const utc = hour.dt * 1000 + localOffset;
        //     hour.currentTime = utc + 1000 * weatherData.timezone_offset;
        // })
        return weatherData.data;
    } catch(err) {
        console.log(err);
    }
};

const weatherData = await getWeatherData();

const router = useRouter();
const removeCity = () => {
    const cities = JSON.parse(localStorage.getItem('savedCities'));
    const updatedCities = cities.filter((city) => city.id !== route.query.id);

    localStorage.setItem('savedCities', JSON.stringify(updatedCities));
    router.push({
        name: 'home',
    });
}; 
</script>
