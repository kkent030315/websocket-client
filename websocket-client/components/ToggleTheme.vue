<template>
  <div class="themeControl">
    <button class="themeControl__button" @click="swapLightAndDarkMode">
      <DarkThemeIcon v-if="isLightMode()" class="themeControl__button__icon" />
      <LightThemeIcon v-else class="themeControl__button__icon" />
    </button>
  </div>
</template>

<script>
import LightThemeIcon from "@/assets/svg/light.svg?inline";
import DarkThemeIcon from "@/assets/svg/dark.svg?inline";

export default {
  components: {
    LightThemeIcon,
    DarkThemeIcon,
  },
  methods: {
    isLightMode() {
      const currentMode = this.$colorMode.preference;
      return currentMode === "light";
    },
    swapLightAndDarkMode() {
      this.$colorMode.preference = this.isLightMode() ? "dark" : "light";
    },
  },
};
</script>

<style lang="scss">
@import "./assets/sass/variables.scss";

.themeControl {
  display: flex;
  flex-direction: row;

  &__button {
    padding: 10px;

    &__icon {
      color: $color-icon;
    }
  }
}
</style>