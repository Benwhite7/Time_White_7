<template>
  <RainCanvasVue :setTime="backGroundPage" />
  <div class="app">
    <h1 class="main-title">Clima 🌤️</h1>

    <SearchSection v-model="city" :askWeather="getWeather" ref="searchRef"/>
    <div class="loading-init" v-if="isLoading">
      <span></span>
      <span :style="{ animationDelay: `250ms` }"></span>
      <span :style="{ animationDelay: `500ms` }"></span>
    </div>
    <p v-if="error" class="error">{{ error }}</p>
    <transition name="fade-city">
      <div class="main-city" v-if="weather">
        <button @click="verifyFavoriteButton(weather.name) ? removeFavoriteCity(weather) : addFavoriteCity(weather)">
          {{ verifyFavoriteButton(weather.name) ? '💗' : '🤍' }}
        </button>
        <h2 class="h2-cities">{{ `${weather.name} (${weather.sys.country})` }}</h2>
        <p class="h2-cities">{{ getLocalTime(weather) }}</p>
        <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`" />
        <p class="weather-description">{{ weather.weather[0].description.toUpperCase() }}</p>
        <p v-if="!windowLong">🌡️ {{ weather.main.temp }} °C</p>
        <section v-if="windowLong" class="extra-info-weather">
          <div>
            <h3>Temperatura</h3>
            <p class="data-weather-main-city">🌡️ {{ weather.main.temp }} °C</p>
          </div>
          <div>
            <h3>Humedad</h3>
            <p class="data-weather-main-city">💧 {{ weather.main.humidity }} %</p>
          </div>
          <div>
            <h3>Velocidad del viento</h3>
            <p class="data-weather-main-city">🍃 {{ weather.wind.speed }} m/s</p>
          </div>
          <div>
            <h3>Direccion del viento</h3>
            <p class="data-weather-main-city">⏫ {{ weather.wind.deg }}°</p>
          </div>
        </section>

      </div>
    </transition>
    <FavoriteCities :cities="favoriteCities" :removeCity="removeFavoriteCity" :calculateTime="getLocalTime"/>
    <CitiesContainer :data="initCitys" :addCity="addFavoriteCity" :removeCity="removeFavoriteCity"
      :favoriteFunc="verifyFavoriteButton" :calculateTime="getLocalTime"/>
  </div>
  <footer class="time-footer">
    <p>🌤️ App del Clima - Desarrollada por Benjamin Melendez© 2025</p>
    <p>Licencia MIT | <a href="https://github.com/Benwhite7">GitHub</a></p>
  </footer>
</template>

<script>
import "./assets/styles/app.css";
import CitiesContainer from "./components/ContainerCities.vue"
import RainCanvasVue from "./components/WeatherCanvas.vue";
import FavoriteCities from "./components/FavoriteCities.vue";
import SearchSection from "./components/SearchSection.vue";

import { ref } from "vue";

export default {
  components: { CitiesContainer, RainCanvasVue, FavoriteCities, SearchSection },
  data() {
    return {
      initCitys: null,
      weather: null,
      error: null,
      searchRef: ref(null),
      city: ref(''),
      searchs: [],
      remenberSearch: [],
      favoriteCities: [],
      saveFavorites: [],
      URLAPI: "https://api.openweathermap.org/data/2.5/",
      backGroundPage: "bg-default",
      apiKey: import.meta.env.VITE_API_KEY,
      windowLong: false,
      isLoading: true,
    };
  },
  mounted() {
    window.addEventListener("resize", this.verifySizeWindow);
    this.windowLong = window.innerWidth > 640;
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const lat = position.coords.latitude;
        const lon = position.coords.longitude;
        this.getWeather({ lat, lon });
      },
      (error) => {
        console.error(error);
      }
    );
    this.getCitiesWeather();
    let itemsFavorites = localStorage.getItem("saveFavorites");
    if (itemsFavorites && itemsFavorites.length > 2) {
      const arrayFavorites = JSON.parse(itemsFavorites);
      this.saveFavorites = arrayFavorites;
      let stringFavorites = arrayFavorites.join(',');
      const consult = `${this.URLAPI}group?id=${stringFavorites}&appid=${this.apiKey}&units=metric&lang=es`;
      this.getCitiesFavorites(consult);
    }
  },

  beforeUnmount() {
    document.removeEventListener("click", this.handleClickOutside);
    window.removeEventListener("resize", this.verifySizeWindow);
  },

  methods: {
      async getWeather(coords) {
        if (!this.city && !coords) return;
        this.error = null;
        const URLBODY = this.city ? `q=${this.city}` : `lat=${coords.lat}&lon=${coords.lon}`;
        const URL = `${this.URLAPI}weather?${URLBODY}&appid=${this.apiKey}&units=metric&lang=es`;
        try {
          const res = await fetch(URL);
          if (!res.ok) throw new Error("Ciudad no encontrada");
          const climaRes = await res.json();
          const { weather: resWeather, name: resName, sys, dt, main } = climaRes;
          this.weather = climaRes;
          this.stablicingBackground(resWeather[0].description, { sys, dt, main });
          this.$refs.searchRef.modifiedOnCallApi(resName);
        } catch (error) {
          this.weather = null;
          this.error = error.message;
        }
      },

    async getCitiesWeather() {
      this.error = null;
      const cities = `${this.URLAPI}group?id=3435910,3469058,3646738,3652462,3688689,3553478,3128760,2643743,5128581,1850147,5368361,3169070,2147714,2950159,1835848,524901,6167865,3448439&appid=${this.apiKey}&units=metric&lang=es`;
      try {
        const res = await fetch(cities);
        if (!res.ok) throw new Error("Ciudad no encontrada");
        let { list } = await res.json();
        list.sort((ca, cb) => ca.name.localeCompare(cb.name));
        this.isLoading = false;
        this.initCitys = list;
      } catch (error) {
        this.isLoading = false;
        this.weather = null;
        this.error = error.message;
      }
    },

    async getCitiesFavorites(URL) {
      this.error = null;
      try {
        const res = await fetch(URL);
        if (!res.ok) throw new Error("No se pueden cargar los favoritos");
        let respuesta = await res.json();
        this.favoriteCities = respuesta.list;
      } catch (error) {
        this.saveFavorites = [];
        this.error = error.message;
      }
    },

    stablicingBackground(resClima, time) {
      let body = document.querySelector("body");
      if (this.backGroundPage) body.classList.remove(this.backGroundPage);
      this.backGroundPage = this.getBackgroundString(resClima, time);
      body.classList.add(this.backGroundPage);
    },

    addFavoriteCity(city) {
      const IHaveCity = this.favoriteCities.some((c) => c.name === city.name);
      if (IHaveCity) return;
      if (this.favoriteCities.length >= 4) return;
      this.favoriteCities.push(city);
      this.saveFavorites.push(city.id);
      let stringFavoritesCities = JSON.stringify(this.saveFavorites);
      localStorage.setItem("saveFavorites", stringFavoritesCities);
    },

    removeFavoriteCity(city) {
      let index;
      index = this.favoriteCities.indexOf(city);
      if (index == -1) index = this.favoriteCities.findIndex((c) => c.name === city.name);
      if (index == -1) return;
      let indexId = this.saveFavorites.indexOf(city.id);
      this.favoriteCities.splice(index, 1);
      this.saveFavorites.splice(indexId, 1);
      let stringFavoritesCities = JSON.stringify(this.saveFavorites);
      localStorage.setItem("saveFavorites", stringFavoritesCities);
    },

    verifyFavoriteButton(name) {
      const existCity = this.favoriteCities.some((c) => c.name === name);
      return existCity;
    },

    getBackgroundString(description, time) {
      const { sys, dt, main } = time;
      let clima = description.toLowerCase();
      const isNight = dt < sys.sunrise || dt > sys.sunset;
      const isSunrise = dt >= sys.sunrise - 1800 && dt <= sys.sunrise + 1800;
      const isSunset = dt >= sys.sunset - 1800 && dt <= sys.sunset + 1800;
      const isHot = main.temp > 28;
      let bgBase;

      if (clima.includes("tormenta")) return "bg-storm";


      if (isSunrise || isSunset) {
        bgBase = "bg-twilight"
      } else if (isNight) {
        bgBase = "bg-night"
      } else {
        bgBase = ""
      }

      if (["cielo claro", "despejado"].includes(clima)) {
        return isHot && !isNight
          ? "-bg-sunny-hot"
          : `${bgBase}-bg-sunny-template`;
      }

      const bgMap = [
        { keywords: ["algo de nubes"], class: "-bg-cloudy-sun" },
        { keywords: ["nubes", "nuboso"], class: "-bg-cloudy" },
        { keywords: ["niebla", "bruma", "humo"], class: "-bg-fog" },
        { keywords: ["lluvia de gran", "chubasco"], class: "-bg-rain-big" },
        { keywords: ["lluvia"], class: "-bg-rain" },
        { keywords: ["nieve"], class: "-bg-snow" },
      ];

      for (const { keywords, class: bgClass } of bgMap) {
        if (keywords.some((k) => clima.includes(k)))
          return `${bgBase}${bgClass}`;
      }

      return "bg-default";
    },

    getLocalTime(c) {
      const TIMEZONE = c.timezone ? c.timezone : c.sys.timezone;
      const localTimeStamp = (c.dt + TIMEZONE) * 1000;
      const localDate = new Date(localTimeStamp);
      return localDate.toUTCString().split(" ")[4].slice(0, 5);
    },

    verifySizeWindow() {
      this.windowLong = window.innerWidth > 640;
    }
  },
};
</script>