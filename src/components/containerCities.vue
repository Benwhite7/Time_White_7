<template>
  <transition-group name="slide" tag="div" class="tarjetas">
    <section class="init-cities" v-if="data && data.length > 0">
      <div class="init-cities-card" v-for="(city, index) in data" :key="index" :style="{ animationDelay: `${index * 150}ms` }">
        <h2 class="h2-cities">{{ city.name }}</h2>
        <h2 class="h2-cities">{{ getLocalTime(city) }}</h2>
        <p>{{ city.weather[0].description.toUpperCase() }}</p>
        <p>🌡️ {{ city.main.temp }} °C</p>
        <img
          :src="`https://openweathermap.org/img/wn/${city.weather[0].icon}@2x.png`"
        />
        <button v-if="!favoriteFunc(city.name)" class="favorite-city-button" @click="addCity(city)">🤍</button>
        <button v-if="favoriteFunc(city.name)" class="favorite-city-button" @click="removeCity(city)">💗</button>
      </div>
    </section>
  </transition-group>
</template>
<script>
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

  },
  methods: {
    getLocalTime(c) {
      const localTimeStamp = (c.dt + c.sys.timezone) * 1000;
      const localDate = new Date(localTimeStamp);
      return localDate.toUTCString().split(' ')[4].slice(0,5);
    }
  }
};
</script>