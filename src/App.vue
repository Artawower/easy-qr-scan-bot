<template>
  <div id="main">
    <div
      v-if="is_telegram_client && is_telegram_api_updated"
      class="text-center"
    >
      <!--<h3>Window Control</h3>
      <b>isExpanded</b>: {{ TWA.isExpanded }}
      <button @click="TWA.expand()">Expand</button>
      <button @click="TWA.close()">Close</button><br>
      <h3>Functions and buttons</h3>
      -->

      <div v-if="code">
        <h3>QR code:</h3>
        {{ code }} <br />

        <v-btn v-if="is_url" size="large" @click="openLink()">
          Open Link
        </v-btn>
        <button @click="copyCodeClipboard()">copy to clipboard</button>
      </div>
      {{ codeAppeared }}

      <div v-if="!code" class="code-wrapper">
        <h3>Scan a QR code!</h3>
        <div class="icon"></div>
      </div>
    </div>

    <div v-if="!is_telegram_client" class="text-center">
      Please open the app from a Telegram client!<br />
    </div>
    <div
      v-if="is_telegram_client && !is_telegram_api_updated"
      class="text-center"
    >
      Please update Telegram to Use the app!<br />
      Telegram API version needed 6.4 or greater.<br />
      Your Telegram API version: {{ TWA.version }}
    </div>
  </div>
</template>

<script>
import { prepareUrl } from "./helpers";

export default {
  data() {
    return {
      is_telegram_client: false,
      is_telegram_api_updated: false,
      code: null,
      is_url: false,
      url: null,
      codeAppeared: false,
    };
  },
  created() {
    // Binding function to all the event types
    //this.TWA.onEvent('themeChanged', this.themeChanged);
    this.TWA.MainButton.setText("Scan QR code");
    this.TWA.onEvent("qrTextReceived", this.processQRCode);
    this.TWA.onEvent("mainButtonClicked", this.mainButtonClicked);

    this.is_telegram_api_updated = this.TWA.isVersionAtLeast("6.4");
    // platform not updated if version is not 6.4 or greater

    if (this.TWA.platform != "unknown") {
      this.is_telegram_client = true;
    }

    if (this.is_telegram_client && this.is_telegram_api_updated) {
      this.TWA.MainButton.show();
      // this.showQRScanner();
    }
  },
  mounted() {
    this.TWA.ready();
  },
  methods: {
    // attached with onEvent function during created
    themeChanged() {
      //this.TWA.showAlert('Theme has changed');
    },
    mainButtonClicked() {
      this.showQRScanner();
    },
    openLink() {
      this.TWA.openLink(this.url);
    },
    processQRCode(data) {
      this.code = data.data;
      const result = prepareUrl(this.code);
      this.is_url = result.is_url;
      this.url = result.value;
      this.hapticImpact();
      this.copyCodeClipboard();

      this.TWA.showAlert(`QR code has recognised
and
copied to the clipboard
`);

      this.TWA.closeScanQrPopup();
      // setTimeout(() => {
      //   this.TWA.closeScanQrPopup();
      //   this.TWA.close();
      // }, 1500);
    },
    // End of callbacks
    showQRScanner() {
      const par = {
        text: "",
      };
      this.TWA.showScanQrPopup(par);
    },
    hapticImpact() {
      // light medium heavy rigid soft
      this.TWA.HapticFeedback.impactOccurred("heavy");
    },
    copyCodeClipboard() {
      var Url = this.$refs.mylink;
      Url.innerHTML = window.location.href;
      console.log(Url.innerHTML);
      Url.select();
      document.execCommand("copy");
    },
  },
  watch: {
    code(val) {
      if (val) {
        this.processQRCode();
        this.codeAppeared = true;
        return;
      }
      this.codeAppeared = false;
    },
  },
};
</script>

<style scoped>
/*
bg_color            .
secondary_bg_color  var(--tg-theme-secondary-bg-color)
link_color          var(--tg-theme-link-color).
*/
#main {
  background-color: var(--tg-theme-bg-color, white);
  color: var(--tg-theme-text-color, black);
  /*https://stackoverflow.com/questions/1165497/how-to-prevent-text-from-overflowing-in-css*/
  word-wrap: break-word;
}
b {
  color: var(--tg-theme-hint-color, black);
}
h3 {
  color: var(--tg-theme-text-color, black);
}
button {
  background-color: var(--tg-theme-button-color, #008cba);
  border: 5px;
  color: var(--tg-theme-button-text-color, black);
  padding: 15px;
  margin: 5px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 15px;
}

.code-wrapper {
  display: flex;
  flex-direction: column;
  gap: 20px;
  justify-content: center;
  align-items: center;
}

.icon {
  width: 200px;
  height: 200px;
  background-image: url("/dick.png");
}
</style>
