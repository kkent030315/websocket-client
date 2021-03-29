<template>
  <header class="header">
    <div class="innerContainer">
      <div class="header-left">
        <WebSocketLogo width="40" height="40" class="header-left__logo" />
        <p class="header-left__siteTitle">WebSocket Client</p>
      </div>
      <div class="header-right">
        <ToggleTheme />
      </div>
    </div>
  </header>
</template>

<script>
import WebSocketLogo from "@/assets/svg/websocket.svg?inline";
import ToggleTheme from "@/components/ToggleTheme.vue";

export default {
  components: {
    WebSocketLogo,
    ToggleTheme,
  },
};
</script>

<style lang="scss">
@import "./assets/sass/variables.scss";

.header {
  width: 100%;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  border-bottom: 1px solid $color-border;

  .innerContainer {
    justify-content: center;
    flex-direction: row;
  }

  &-left,
  &-right {
    display: flex;
    flex-direction: row;
    align-items: center;
  }

  &-left {
    justify-content: flex-start;

    &__logo {
      color: $color-logo;
      fill: $color-logo;
      margin: 10px;
    }

    &__siteTitle {
      white-space: nowrap;
      font-size: 1.2rem;
    }
  }

  &-right {
    width: 100%;
    justify-content: flex-end;
  }
}
</style>