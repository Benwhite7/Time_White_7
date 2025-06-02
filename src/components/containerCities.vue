<template>
  <TransitionGroup name="slide" tag="div" class="tarjetas">
      <div class="init-cities-card" v-for="(city, index) in data" :key="city.name" :style="{ animationDelay: `${index * 250}ms` }">
        <h2 class="h2-cities">{{ city.name }}</h2>
        <h2 class="h2-cities">{{ calculateTime(city) }}</h2>
        <p>{{ city.weather[0].description.toUpperCase() }}</p>
        <p>🌡️ {{ city.main.temp }} °C</p>
        <img
          :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@2x.png`"
        />
        <button class="favorite-city-button" @click="favoriteFunc(city.name) ? removeCity(city) : addCity(city)">
          {{ favoriteFunc(city.name) ? '💗' : '🤍' }}
        </button>
      </div> 
  </TransitionGroup>
</template>
<script>
import { TransitionGroup } from "vue";
import "../assets/styles/cities_container.css";
export default {
  name: "CitiesContainer",
  props: {
    data: {
      type: Array,
      default: () => []
    },
    addCity: {
      type: Function,
      required: true
    },
    removeCity: {
      type: Function,
      required: true
    },
    favoriteFunc: {
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