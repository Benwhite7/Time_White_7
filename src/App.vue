<template>
  <RainCanvasVue :setTime="backGroundPage" />
  <div class="app">
    <h1>Clima 🌤️</h1>

    <section class="interactive-section">
      <div>
        <input @mouseenter="showHistorial = true" @focus="showHistorial = true" @keyup.enter="getWeather"
          @input="pushSuggestions" v-model="city" placeholder="Ingresa una ciudad" />

        <transition name="fade">
          <article v-if="showHistorial" class="history-searchs-container" ref="searchs">
            <div class="history-search-card" v-for="(search, index) in searchs" :key="index">
              <option @click="changeInput(search)" :value="search">
                {{ search }}
              </option>
              <button class="button_delete_search_card" @click="deleteSearchCard(search)">
                ❌
              </button>
            </div>
          </article>
        </transition>
      </div>

      <button class="interactive-section_button" @click="getWeather">
        Buscar
      </button>
    </section>

    <p v-if="error" class="error">{{ error }}</p>
    <transition name="fade-city">
      <div class="main-city" v-if="weather">
        <button v-if="verifyFavoriteButton(weather.name)" @click="removeFavoriteCity(weather)">💗</button>
        <button v-if="!verifyFavoriteButton(weather.name)" @click="addFavoriteCity(weather)">🤍</button>
        <h2 class="h2-cities">{{ weather.name }}</h2>
        <p class="h2-cities">{{ getLocalTime(weather) }}</p>
        <p>{{ weather.weather[0].description.toUpperCase() }}</p>
        <p>🌡️ {{ weather.main.temp }} °C</p>
        <img :src="`https://openweathermap.org/img/wn/${weather.weather[0].icon}@2x.png`" />
      </div>
    </transition>
    <FavoriteCities :cities="favoriteCities" :removeCity="removeFavoriteCity" />
    <CitiesContainer :data="initCitys" :addCity="addFavoriteCity" :removeCity="removeFavoriteCity"
      :favoriteFunc="verifyFavoriteButton" />
  </div>
</template>

<script>
import "./assets/styles/app.css";
import CitiesContainer from "./components/containerCities.vue";
import RainCanvasVue from "./components/RainCanvas.vue";
import FavoriteCities from "./components/favoriteCities.vue";

export default {
  components: { CitiesContainer, RainCanvasVue, FavoriteCities },
  data() {
    return {
      initCitys: null,
      weather: null,
      error: null,
      searchs: [],
      city: "",
      favoriteCities: [],
      saveFavorites: [],
      showHistorial: false,
      URLAPI: "https://api.openweathermap.org/data/2.5/",
      backGroundPage: "bg-default",
    };
  },
  created() {
    this.getCitiesWeather();
  },
  mounted() {
    document.addEventListener("click", this.handleCLickOutside);
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
    let itemsFavorites = localStorage.getItem("saveFavorites");
    if (itemsFavorites) {
      const arrayFavorites = JSON.parse(itemsFavorites);
      this.saveFavorites = arrayFavorites;
      let stringFavorites = arrayFavorites.join(',');
      const apiKey = import.meta.env.VITE_API_KEY;
      const consult = `${this.URLAPI}group?id=${stringFavorites}&appid=${apiKey}&units=metric&lang=es`;
      this.getCitiesFavorites(consult);
    }
  },

  beforeUnmount() {
    document.removeEventListener("click", this.handleClickOutside);
  },

  methods: {
    async getWeather(coords) {
      if (!this.city && !coords) return;
      this.error = null;
      const apiKey = import.meta.env.VITE_API_KEY;
      const URLBODY = this.city
        ? `q=${this.city}`
        : `lat=${coords.lat}&lon=${coords.lon}`;
      const URL = `${this.URLAPI}weather?${URLBODY}&appid=${apiKey}&units=metric&lang=es`;
      try {
        const res = await fetch(URL);
        if (!res.ok) throw new Error("Ciudad no encontrada");
        const climaRes = await res.json();
        this.weather = climaRes;
        this.stablicingBackground(climaRes.weather[0].description, climaRes);
        if (this.searchs.length >= 4) this.searchs.pop();
        this.verifyExistSearch(climaRes.name);
        this.searchs.unshift(climaRes.name);
        let historyJSON = JSON.stringify(this.searchs);
        localStorage.setItem("historySearchs", historyJSON);
      } catch (error) {
        this.weather = null;
        this.error = error.message;
      }
    },

    async getCitiesWeather() {
      this.error = null;
      let history_Searchs = localStorage.getItem("historySearchs");
      if (!history_Searchs) localStorage.setItem("historySearchs", []);
      this.searchs = JSON.parse(history_Searchs);
      const apiKey = import.meta.env.VITE_API_KEY;
      const cities = `${this.URLAPI}group?id=3435910,3469058,3646738,3652462,3688689,3936456,3553478,3128760,2643743,5128581,1850147,5368361,3169070,3530597,2147714,2950159,1835848,524901,6167865,3448439&appid=${apiKey}&units=metric&lang=es`;
      try {
        const res = await fetch(cities);
        if (!res.ok) throw new Error("Ciudad no encontrada");
        let { list } = await res.json();
        this.initCitys = list;
      } catch (error) {
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
      const codeBackGround = this.setBackgroundPage(resClima, time);
      this.backGroundPage = codeBackGround;
      body.classList.add(codeBackGround);
    },

    changeInput(valueOne) {
      this.city = valueOne;
      this.showHistorial = false;
    },

    verifyExistSearch(search) {
      if (this.searchs.includes(search)) {
        let indexFound = this.searchs.indexOf(search);
        this.searchs.splice(indexFound, 1);
      }
    },

    deleteSearchCard(city) {
      let indexFound = this.searchs.indexOf(city);
      this.searchs.splice(indexFound, 1);
      let historySearchJSON = JSON.stringify(this.searchs);
      localStorage.setItem("historySearchs", historySearchJSON);
    },

    async pushSuggestions() {
      if (this.city.length < 3) return;
      const apiKey = import.meta.env.VITE_API_KEY;
      try {
        const res = await fetch(
          `https://api.openweathermap.org/geo/1.0/direct?q=${this.city}&limit=5&appid=${apiKey}`
        );
        const response = await res.json();
        console.log(response);
      } catch (error) {
        console.error(error.message);
      }
    },

    addFavoriteCity(city) {
      const IHaveCity = this.favoriteCities.some((c) => c.name === city.name);
      if (IHaveCity) return;
      if (this.favoriteCities.length >= 4) return;
      this.favoriteCities.push(city);
      console.log(city);
      this.saveFavorites.push(city.id);
      let stringFavoritesCities = JSON.stringify(this.saveFavorites);
      localStorage.setItem("saveFavorites", stringFavoritesCities);
    },

    removeFavoriteCity(city) {
      console.log(this.favoriteCities);
      console.log(city);
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

    handleCLickOutside(event) {
      const elementInQuestion = this.$refs.searchs;
      if (elementInQuestion && !elementInQuestion.contains(event.target)) {
        this.showHistorial = false;
      }
    },

    setBackgroundPage(description, time) {
      let clima = description.toLowerCase();
      const isNight = time.dt < time.sys.sunrise || time.dt > time.sys.sunset;
      const isHot = time.main.temp > 28;

      if (clima.includes("tormenta")) return "bg-storm";

      let bgBase = isNight ? "bg-night" : "";

      if (["cielo claro", "despejado"].includes(clima)) {
        return isHot && !isNight
          ? "-bg-sunny-hot"
          : `${bgBase}-bg-sunny-template`;
      }

      const bgMap = [
        { keywords: ["algo de nubes"], class: "-bg-cloudy-sun" },
        { keywords: ["nubes", "nuboso"], class: "-bg-cloudy" },
        { keywords: ["niebla", "bruma"], class: "-bg-fog" },
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
      const localTimeStamp = (c.dt + c.timezone) * 1000;
      const localDate = new Date(localTimeStamp);
      return localDate.toUTCString().split(" ")[4].slice(0, 5);
    },
  },
};
</script>