<template>
  <section class="favorite-cities" v-if="cities && cities.length > 0">
    <div class="favorite-city" v-for="(city, index) in cities" :key="index">
      <h2>{{ city.name }}</h2>
      <p>{{ getLocalTime(city) }}</p>
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
    }
  },
  methods: {
    getLocalTime(c) {
      console.log(c);
      const TIMEZONE = c.timezone ? c.timezone : c.sys.timezone;
      const localTimeStamp = (c.dt + TIMEZONE) * 1000;
      const localDate = new Date(localTimeStamp);
      return localDate.toUTCString().split(" ")[4].slice(0, 5);
    },
  },
};
</script>