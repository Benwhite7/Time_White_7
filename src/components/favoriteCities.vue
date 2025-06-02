<template>  
  <section class="favorite-cities" v-if="cities && cities.length > 0">    
    <div class="favorite-city" v-for="(city, index) in cities" :key="index">
      <h2 class="h2-cities">{{ city.name }}</h2>
      <p class="h2-cities">{{ calculateTime(city) }}</p>
      <p>{{ city.weather[0].description.toUpperCase() }}</p>
      <p>🌡️ {{ city.main.temp }} °C</p>
      <img
        :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@2x.png`"
      />
      <button class="favorite-city-button" @click="removeCity(city)">❌</button>
    </div>
  </section>
</template>
<script>
import '../assets/styles/favoriteCities.css';
export default {
  name: "FavoriteCities",
  props: {
    cities: {
      type: Array,
      default: () => [],
    },
    removeCity: {
      type: Function,
      required: true
    },
    calculateTime:{
      type: Function,
      required: true
    }
  }
};
</script>