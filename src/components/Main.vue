<template>
  <main class="main">
    <div class="container">
      <section class="main__box">
        <div class="main__box-left">
          <div class="main__left-info">
            <div class="main__left-temp">
              <h2 class="main__left-degree">{{ showTemp() }}°</h2>
              <h2 class="main__left-data">{{ getWeekDayLabel() }}</h2>
            </div>
            <div class="main__left-icon">
              <img :src="getWeatherIcon()" alt="" class="main__left-iconImg" />
            </div>
          </div>
          <div class="main__left-details">
            <h2 class="main__left-time">Время: {{ getLocalTime() }}</h2>
            <h2 class="main__left-city">Город: {{ showCityName() }}</h2>
          </div>
        </div>
        <div class="main__box-right">
          <img class="main__right-bg" src="@/assets/images/cloud.png" alt="" />
          <div class="main__right-info">
            <div class="main__right-item">
              <img
                class="main__right-img"
                src="@/assets/icons/temp.svg"
                alt=""
              />
              <h2 class="main__right-title">Температура</h2>
              <p class="main__right-text">
                {{ showTemp() }}° - ощущается как {{ showTempFeelsLike() }}°
              </p>
            </div>
            <div class="main__right-item">
              <img
                class="main__right-img"
                src="@/assets/icons/pressure.svg"
                alt=""
              />
              <h2 class="main__right-title">Давление</h2>
              <p class="main__right-text">
                {{ showPressure() }}
              </p>
            </div>
            <div class="main__right-item">
              <img
                class="main__right-img"
                src="@/assets/icons/precipitation.svg"
                alt=""
              />
              <h2 class="main__right-title">Осадки</h2>
              <p class="main__right-text">{{ showPrecipitation() }}</p>
            </div>
            <div class="main__right-item">
              <img
                class="main__right-img"
                src="@/assets/icons/wind.svg"
                alt=""
              />
              <h2 class="main__right-title">Ветер</h2>
              <p class="main__right-text">{{ showWind() }}</p>
            </div>
          </div>
        </div>
      </section>
      <Transition name='fade' mode="out-in">
      <section class="main__weather" v-show="forecast">
        <div
          class="main__weather-item"
          v-for="day in getDailyForecast()"
          :key="day.dt_txt"
        >
          <div class="main__weather-top">
            <h2 class="main__top-data">{{ getWeekDay(day.dt_txt) }}</h2>
            <p class="main__top-num">{{ formatDate(day.dt_txt) }}</p>
          </div>

          <img
            class="main__weather-img"
            :src="getWeatherIconByCode(day.weather[0].icon)"
            alt=""
          />

          <div class="main__weather-down">
            <p class="main__down-degree">{{ Math.round(day.main.temp) }}°</p>
            <p class="main__down-shade">
              {{ Math.round(day.main.feels_like) }}°
            </p>
            <p class="main__down-cloud">{{ day.weather[0].description }}</p>
          </div>
        </div>
      </section>
      </Transition>
    </div>
  </main>
</template>

<script>
export default {
  props: {
    city: {
      type: String,
    },
    info: {
      type: Object,
    },
    forecast: {
      type: Object,
    },
    currentCity: {
      type: String,
    },
  },
  methods: {
    showTemp() {
      return this.info?.main?.temp ? Math.round(this.info.main.temp) : "00";
    },
    showTempFeelsLike() {
      return this.info?.main?.feels_like
        ? Math.round(this.info.main.feels_like)
        : "00";
    },
    showCityName() {
      return this.info?.name ? this.info.name : "Введите город";
    },
    getLocalTime() {
      if (!this.info || !this.info.dt || !this.info.timezone) return "00:00";

      const localTimestamp = (this.info.dt + this.info.timezone) * 1000; // ms
      const date = new Date(localTimestamp);
      const hours = date.getUTCHours().toString().padStart(2, "0");
      const minutes = date.getUTCMinutes().toString().padStart(2, "0");

      return `${hours}:${minutes}`;
    },
    getWeekDayLabel() {
      if (!this.info || !this.info.dt || !this.info.timezone) return "День";

      const current = new Date();
      const cityDate = new Date((this.info.dt + this.info.timezone) * 1000);

      // Сравнение по дню, месяцу и году
      const isToday =
        current.getUTCDate() === cityDate.getUTCDate() &&
        current.getUTCMonth() === cityDate.getUTCMonth() &&
        current.getUTCFullYear() === cityDate.getUTCFullYear();

      if (isToday) {
        return "Сегодня";
      }

      const days = [
        "Воскресенье",
        "Понедельник",
        "Вторник",
        "Среда",
        "Четверг",
        "Пятница",
        "Суббота",
      ];

      return days[cityDate.getUTCDay()];
    },
    showPressure() {
      if (!this.info?.main?.pressure) return "Нет данных";
      const hPa = this.info.main.pressure; // гПа
      const mmHg = Math.round(hPa * 0.75006); // перевод в мм рт. ст.
      return `${mmHg} миллиметры ртутного столба`;
    },
    showWind() {
      if (!this.info?.wind) return "Нет данных";
      const speed = this.info.wind.speed;
      const deg = this.info.wind.deg;

      const directions = [
        "север",
        "северо-восток",
        "восток",
        "юго-восток",
        "юг",
        "юго-запад",
        "запад",
        "северо-запад",
      ];
      const index = Math.round(deg / 45) % 8;
      const direction = directions[index];

      return `${speed} м/с, ${direction}`;
    },
    showPrecipitation() {
      if (!this.info) return "Нет данных";

      const rain = this.info.rain?.["1h"] || 0;
      const snow = this.info.snow?.["1h"] || 0;

      if (rain > 0) {
        return `Дождь: ${rain} мм`;
      } else if (snow > 0) {
        return `Снег: ${snow} мм`;
      } else {
        return "Без осадков";
      }
    },
    getWeatherIcon() {
      if (!this.info?.weather?.[0]?.icon) {
        return new URL("/src/assets/icons/sun.svg", import.meta.url).href;
      }

      const iconCode = this.info.weather[0].icon;

      const iconMap = {
        "01d": "sun.svg",
        "01n": "sun.svg",
        "02d": "mainly_cloudy.svg",
        "02n": "mainly_cloudy.svg",
        "03d": "mainly_cloudy.svg",
        "03n": "mainly_cloudy.svg",
        "04d": "mainly_cloudy.svg",
        "04n": "mainly_cloudy.svg",
        "09d": "rain.svg",
        "09n": "rain.svg",
        "10d": "small_rain_sun.svg",
        "10n": "small_rain.svg",
        "11d": "rain.svg",
        "11n": "rain.svg",
        "13d": "precipitation.svg",
        "13n": "precipitation.svg",
        "50d": "mainly_cloudy.svg",
        "50n": "mainly_cloudy.svg",
      };

      const fileName = iconMap[iconCode] || "sun.svg";
      return new URL(`/src/assets/icons/${fileName}`, import.meta.url).href;
    },
    getDailyForecast() {
      if (!this.forecast?.list) return [];

      const map = new Map();

      for (const entry of this.forecast.list) {
        const date = entry.dt_txt.split(" ")[0];
        if (!map.has(date)) {
          map.set(date, entry);
        }
      }

      return [...map.values()].slice(0, 7); // 7 дней
    },
    getWeekDay(dateStr) {
      const days = [
        "Воскресенье",
        "Понедельник",
        "Вторник",
        "Среда",
        "Четверг",
        "Пятница",
        "Суббота",
      ];
      return days[new Date(dateStr).getDay()];
    },
    formatDate(dateStr) {
      const date = new Date(dateStr);
      const options = { day: "numeric", month: "short" };
      return date.toLocaleDateString("ru-RU", options); // например, "5 июл"
    },
    getWeatherIconByCode(code) {
      const iconMap = {
        "01d": "sun.svg",
        "01n": "sun.svg",
        "02d": "mainly_cloudy.svg",
        "02n": "mainly_cloudy.svg",
        "03d": "mainly_cloudy.svg",
        "03n": "mainly_cloudy.svg",
        "04d": "mainly_cloudy.svg",
        "04n": "mainly_cloudy.svg",
        "09d": "rain.svg",
        "09n": "rain.svg",
        "10d": "small_rain_sun.svg",
        "10n": "small_rain.svg",
        "11d": "rain.svg",
        "11n": "rain.svg",
        "13d": "precipitation.svg",
        "13n": "precipitation.svg",
        "50d": "mainly_cloudy.svg",
        "50n": "mainly_cloudy.svg",
      };

      const fileName = iconMap[code] || "sun.svg";
      return new URL(`/src/assets/icons/${fileName}`, import.meta.url).href;
    },
  },
};
</script>

<style>
</style>