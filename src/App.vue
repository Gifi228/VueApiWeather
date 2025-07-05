<template>
  <div :class="{ dark: isDarkTheme }">
    <Header 
      v-model:city="city" 
      :getWeather="getWeather" 
      @toggle-theme="toggleTheme"
    />
    <Main 
      :city="city" 
      :info="info" 
      :forecast="forecast" 
    />
  </div>
</template>

<script>
import axios from "axios";

import Header from "@/components/Header.vue";
import Main from "@/components/Main.vue";
export default {
  components: {
    Header,
    Main,
  },
  data() {
    return {
      city: "",
      info: null,
      forecast: null,
      currentCity: "",
      error: "",
      isDarkTheme: false,
    };
  },
  watch: {
    isDarkTheme(val) {
      document.body.className = val ? 'light' : 'dark';
    },
  },
  mounted() {
    document.body.className = this.isDarkTheme ? 'light' : 'dark';
  },
  methods: {
    async getWeather() {
      if (this.city.trim().length < 2) {
        this.error = "Нужно название города!";
        alert(this.error);
        return;
      }

      try {
        const [current, forecast] = await Promise.all([
          axios.get("https://api.openweathermap.org/data/2.5/weather", {
            params: {
              q: this.city,
              units: "metric",
              appid: "3d9de74844d28377e81415151cbe6a66",
              lang: "ru",
            },
          }),
          axios.get("https://api.openweathermap.org/data/2.5/forecast", {
            params: {
              q: this.city,
              units: "metric",
              appid: "3d9de74844d28377e81415151cbe6a66",
              lang: "ru",
            },
          }),
        ]);

        this.info = current.data;
        this.forecast = forecast.data;
        this.currentCity = this.city;
        this.error = "";
      } catch (error) {
        this.error = error.response?.data?.message || error.message;
        alert(this.error);
      }
    },
    toggleTheme() {
      this.isDarkTheme = !this.isDarkTheme;
    },
  },
};
</script>

<style>
</style>