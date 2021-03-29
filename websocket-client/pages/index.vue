<template>
  <div class="innerContainer">
    <div class="section">
      <div class="section-inner">
        <table class="globalStatus" cellspacing="0">
          <thead>
            <tr>
              <th>Connection</th>
              <th>Bytes Sent</th>
              <th>Bytes Received</th>
              <th>Elapsed</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td
                class="globalStatus-TextBig"
                :class="
                  connectionState === `connected`
                    ? `online`
                    : connectionState === `error`
                    ? `error`
                    : `offline`
                "
              >
                ●
                {{
                  connectionState === `connected`
                    ? `CONNECTED`
                    : connectionState === `error`
                    ? `ERROR`
                    : `READY`
                }}
              </td>
              <td class="globalStatus-TextBig">{{ bytesSent }}</td>
              <td class="globalStatus-TextBig">
                {{ bytesReceived }}
              </td>
              <td class="globalStatus-TextBig">
                {{ elapsedTime }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div class="connControls">
      <input
        v-model="sockUrl"
        class="normalInput connControls__urlInput"
        type="text"
        placeholder="ws://127.0.0.1:1300"
      />
      <button
        v-if="connectionState !== `connected`"
        class="normalButton primary connControls__performConnect"
        @click="openConnection"
      >
        CONNECT
      </button>
      <button
        v-else-if="connectionState === `connected`"
        class="normalButton error connControls__performConnect"
        @click="closeConnection"
      >
        CLOSE
      </button>
    </div>

    <div class="connIOs">
      <div class="connIOs-sendRequest">
        <textarea
          v-model="sendRequestPayload"
          class="normalInput connIOs-sendRequest__payload"
          cols="30"
          rows="5"
        ></textarea>
        <button class="normalButton primary connIOs-sendRequest__submit">
          Send
        </button>
        <!--
        TODO: Implement sent history
        <div class="section">
          <div class="section-inner"></div>
        </div>
        -->
      </div>
      <div class="connIOs-receivedOutputs">
        <div class="section">
          <div class="section-inner">
            <div class="connIOs-receivedOutputs-control">
              <div
                class="connIOs-receivedOutputs-control__icon"
                @click="clearReceivedOutputs"
              >
                <TrashIcon
                  class="connIOs-receivedOutputs-control__icon__trash"
                />
              </div>
              <div
                class="connIOs-receivedOutputs-control__icon"
                @click="toggleAutoScrollEnabled"
              >
                <ArrowDownIcon
                  :class="isAutoScrollEnabled ? `active` : ``"
                  class="connIOs-receivedOutputs-control__icon__arrowDown"
                />
              </div>
              <div
                class="connIOs-receivedOutputs-control__icon"
                @click="toggleTimestampEnabled"
              >
                <ClockIcon
                  :class="isTimestampEnabled ? `active` : ``"
                  class="connIOs-receivedOutputs-control__icon__arrowDown"
                />
              </div>
            </div>
            <ul
              id="received-outputs"
              class="connIOs-receivedOutputs__outputList"
            >
              <li
                class="connIOs-receivedOutputs__outputList__item"
                v-for="(message, index) in receivedOutputs"
                :key="message.id"
              >
                {{ message }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from "jquery";
import moment from "moment";
import "moment-duration-format";

import TrashIcon from "@/assets/svg/trash.svg?inline";
import ArrowDownIcon from "@/assets/svg/arrow_down.svg?inline";
import ClockIcon from "@/assets/svg/clock.svg?inline";

export default {
  components: {
    TrashIcon,
    ArrowDownIcon,
    ClockIcon,
  },
  data() {
    return {
      connectionState: "init",
      sockUrl: "",
      sockConn: null,

      sentRequests: [],
      receivedOutputs: [],

      bytesSentCount: 0,
      bytesSent: "-",
      bytesReceivedCount: 0,
      bytesReceived: "-",
      elapsedSeconds: 0,
      elapsedTime: "-",
      elapsedIntervalTimer: null,

      isAutoScrollEnabled: true,
      isTimestampEnabled: false,
      sendRequestPayload: "",
    };
  },
  mounted() {
    this.loadSockUrlFromCache();
  },
  methods: {
    loadSockUrlFromCache() {
      if (window) {
        const cachedSockUrl = window.localStorage.getItem(`sockUrl`);
        if (cachedSockUrl) {
          this.sockUrl = cachedSockUrl;
        }
      }
    },
    cacheSockUrl() {
      if (!this.sockUrl) {
        return;
      }

      if (window) {
        window.localStorage.setItem(`sockUrl`, this.sockUrl);
      }
    },
    onConnectFailed() {
      this.connectionState = "error";
    },
    onConnectSuccess() {
      this.connectionState = "connected";
      this.cacheSockUrl();

      this.elapsedIntervalTimer = setInterval(() => {
        const timeFormat = moment
          .duration({ seconds: this.elapsedSeconds })
          .format(`mm:ss`);
        this.elapsedTime =
          this.elapsedSeconds <= 60 ? `00:${timeFormat}` : timeFormat;
        this.elapsedSeconds++;
      }, 1000);
    },
    onClose() {
      this.connectionState = "closed";

      if (this.elapsedIntervalTimer) {
        clearInterval(this.elapsedIntervalTimer);
        this.elapsedIntervalTimer = null;
      }

      this.bytesSentCount = 0;
      this.bytesSent = "-";
      this.bytesReceivedCount = 0;
      this.bytesReceived = "-";
      this.elapsedSeconds = 0;
      this.elapsedTime = "-";
      this.elapsedIntervalTimer = null;
    },
    openConnection() {
      if (!this.sockUrl) {
        return;
      }

      this.sockConn = new WebSocket(`${this.sockUrl}`);

      /**
       * Register callbacks
       */
      this.sockConn.addEventListener(`error`, this.onConnectFailed);
      this.sockConn.addEventListener(`open`, this.onConnectSuccess);
      this.sockConn.addEventListener(`message`, this.onMessage);
      this.sockConn.addEventListener(`close`, this.onClose);
    },
    closeConnection() {
      if (!this.sockConn) {
        return;
      }

      this.sockConn.close();
    },
    onMessage(message) {
      this.bytesReceivedCount += this.lengthInUtf8Bytes(message.data);
      this.bytesReceived = this.formatBytes(this.bytesReceivedCount);

      const formatDate = this.isTimestampEnabled
        ? moment().format(`hh:mm`)
        : "";

      this.receivedOutputs.push(`${formatDate} ${message.data}`);

      if (this.isAutoScrollEnabled) {
        this.scrollSmoothToBottom(`received-outputs`);
      }
    },
    scrollSmoothToBottom(id) {
      const $el = $(`#${id}`)[0];
      if ($el.scrollHeight && $el.clientHeight) {
        $(`#${id}`).animate(
          {
            scrollTop: $el.scrollHeight - $el.clientHeight,
          },
          500
        );
      }
    },
    sendMessage() {
      if (!this.sockConn || !this.sendRequestPayload) {
        return;
      }

      this.sockConn.send(this.sendRequestPayload);

      this.bytesSentCount += this.lengthInUtf8Bytes(this.sendRequestPayload);
      this.bytesSent = this.formatBytes(this.bytesSentCount);
    },
    clearReceivedOutputs() {
      this.receivedOutputs = [];
    },
    lengthInUtf8Bytes(str) {
      /**
       * Matches only the 10.. bytes that are non-initial
       * characters in a multi-byte sequence.
       */
      var m = encodeURIComponent(str).match(/%[89ABab]/g);
      return str.length + (m ? m.length : 0);
    },
    formatBytes(bytes, decimals = 2) {
      if (bytes === 0) return "0 Bytes";

      const k = 1024;
      const dm = decimals < 0 ? 0 : decimals;
      const sizes = ["Bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];

      const i = Math.floor(Math.log(bytes) / Math.log(k));

      return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + " " + sizes[i];
    },
    toggleAutoScrollEnabled() {
      this.isAutoScrollEnabled = !this.isAutoScrollEnabled;
    },
    toggleTimestampEnabled() {
      this.isTimestampEnabled = !this.isTimestampEnabled;
    },
  },
};
</script>

<style lang="scss">
@import "./assets/sass/variables.scss";

.main {
  align-items: center;

  .innerContainer {
    flex-direction: column;
  }
}

.section {
  width: 100%;
  background: $color-background-inner;
  border-radius: 15px;
  margin-bottom: 20px;

  &-inner {
    padding: 20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
}

.globalStatus {
  th,
  td {
    padding: 10px 30px;
    border-right: 1px solid $color-border-inner;
    &:last-child {
      border-right: 0;
    }
  }

  th {
    padding-bottom: 0;
    text-align: left;
    font-family: $font-family-console;
    font-weight: normal;
  }

  td {
    padding-top: 0;
  }

  .online {
    color: $color-status-online;
  }

  .offline {
    color: $color-status-offline;
  }

  .error {
    color: $color-error;
  }

  &-TextMedium {
    font-size: 1.3rem;
  }

  &-TextBig {
    font-size: 2rem;
  }
}

.connControls {
  width: 100%;
  display: flex;
  flex-direction: row;
  margin-bottom: 20px;

  &__urlInput {
    width: 100%;
    margin-right: 10px;
  }
}

.connIOs {
  width: 100%;
  display: flex;
  flex: 1 auto;
  flex-direction: row;

  .section {
    height: 100%;
    max-height: 500px;
  }

  &-sendRequest,
  &-receivedOutputs {
    width: 50%;
    display: flex;
    flex: 1 auto;
    flex-direction: column;
  }

  &-sendRequest {
    margin-right: 20px;

    &__payload {
      margin-bottom: 10px;
    }

    &__submit {
      margin-bottom: 20px;
    }
  }

  &-receivedOutputs {
    &-control {
      width: 100%;
      display: flex;
      flex-direction: row;
      align-items: center;
      margin-bottom: 10px;
      padding-bottom: 10px;
      border-bottom: 1px solid $color-border-inner;

      &__icon {
        padding: 5px;
        border-radius: 50%;
        display: flex;
        flex-direction: row;
        cursor: pointer;
        transition: 0.4s ease background;
        margin-right: 10px;

        &:last-child {
          margin-right: 0;
        }

        &:hover {
          background: $color-background;
          > * {
            fill: $color-primary;
          }
        }

        &__trash,
        &__arrowDown {
          fill: $color-icon-thin;
        }

        .active {
          fill: $color-primary;
        }
      }
    }

    .section-inner,
    &__outputList {
      max-height: 100%;
    }

    &__outputList {
      width: 100%;
      list-style: none;
      display: flex;
      flex-direction: column;
      overflow-y: scroll;

      &::-webkit-scrollbar {
        display: none;
      }

      &__item {
        font-weight: normal;
        word-break: break-all;
        line-height: 1.2;
        margin-bottom: 10px;
        padding-bottom: 10px;
        border-bottom: 2px solid $color-border-inner;

        &:last-child {
          border: none;
          margin-bottom: 0;
          padding-bottom: 0;
        }
      }
    }
  }
}
</style>
