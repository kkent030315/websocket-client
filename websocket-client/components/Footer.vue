<template>
  <footer class="footer">
    <div class="innerContainer">
      <div class="footer-left">
        <p class="footer-left__license">
          <a
            class="footer-left__license__link"
            :href="gitRepoLicenseLink"
            target="_blank"
            >websocket-client</a
          >
          The MIT License.
        </p>
      </div>
      <div class="footer-right">
        <ul class="footer-right-extLinks">
          <li class="footer-right-extLinks__linkItem">
            <p style="margin-right: 10px">View full source at</p>
            <a :href="gitRepoLink" target="_blank">
              <GithubIcon class="footer-right-extLinks__linkItem__icon__git" />
            </a>
          </li>
        </ul>
      </div>
    </div>
  </footer>
</template>

<script>
import GithubIcon from "@/assets/svg/github.svg?inline";

export default {
  components: {
    GithubIcon,
  },
  data() {
    return {
      gitRepoLink: "https://github.com/kkent030315/websocket-client",
      gitRepoLicenseLink:
        "https://github.com/kkent030315/websocket-client/blob/main/LICENSE",
    };
  },
};
</script>

<style lang="scss">
@import "./assets/sass/variables.scss";

.footer {
  width: 100%;
  padding: 20px 0;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  border-top: 1px solid $color-border;

  .innerContainer {
    display: flex;
    flex-direction: row;
  }

  &-left,
  &-right {
    display: flex;
    flex-direction: row;
  }

  &-left {
    &__license {
      white-space: nowrap;
      font-size: 0.9rem;

      &__link {
        color: $color-font-link;
      }
    }
  }

  &-right {
    width: 100%;
    justify-content: flex-end;

    &-extLinks {
      list-style: none;
      display: flex;
      flex-direction: row;

      &__linkItem {
        display: flex;
        justify-content: center;
        align-items: center;

        &__icon {
          color: $color-icon;

          &__git {
            > path {
              fill: $color-icon;
            }
          }
        }
      }
    }
  }
}
</style>