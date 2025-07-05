<template>
  <header class="header">
    <div class="container">
      <nav class="header__nav">
        <div class="header__nav-left">
          <a href="#" class="logo">
            <img src="@/assets/images/logo.png" alt="" />
          </a>
          <h1 class="header__nav-title">Vue Weather</h1>
        </div>
        <div class="header__nav-right">
          <button class="header__nav-switch" @click="$emit('toggle-theme')">
            <img src="@/assets/icons/themeSwitch.svg" alt="" />
          </button>
          <input
            class="header__nav-select"
            :value="city"
            @input="$emit('update:city', $event.target.value)"
            placeholder="Введите Город..."
          />
          <button
            class="header__nav-send"
            v-show="city != ''"
            @click="getWeather() && clearCity()"
          >
            <img src="@/assets/icons/send.svg" alt="" />
          </button>
        </div>
      </nav>
    </div>
  </header>
</template>

<script>
export default {
  props: {
    city: {
      type: String,
      required: true,
    },
    getWeather: {
      type: Function,
      required: true,
    },
  },
  emits: ["update:city"],
  methods: {
    clearCity() {
      this.$emit("update:city", "");
    },
  },
};
</script>

<style>
</style>
