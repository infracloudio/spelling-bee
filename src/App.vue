<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import Hive from "./components/Hive.vue";
import CorrectGuesses from "./components/CorrectGuesses.vue";
import Progress from "./components/Progress.vue";
import YesterdaysAnswers from "./components/YesterdaysAnswers.vue";
import Info from "./components/Info.vue";
import GameWon from "./components/GameWon.vue";
import allAnswers from "../data/allAnswers.json";
import { useMainStore } from "./store";
import { Aim } from "@element-plus/icons-vue";
import { InfoFilled, Calendar, Sunny, Moon } from "@element-plus/icons-vue";

const store = useMainStore();
const showYesterdaysAnswers = ref(false);
const showInfo = ref(false);
const zindex = ref(0);
const gameWonModalShown = ref(false); // only show gameWon modal once
let timer: any;

const fullName = ref("");
const email = ref("");
const popupModalShown = ref(
  !localStorage.getItem("full_name") || !localStorage.getItem("email")
);

const darkmode = ref(store.theme === "dark");
const onToggleDarkMode = () => {
  popupModalShown.value =
    !localStorage.getItem("full_name") || !localStorage.getItem("email");
  if (darkmode.value === true) {
    store.theme = "dark";
    document.documentElement.classList.add("dark");
  } else {
    store.theme = "light";
    document.documentElement.classList.remove("dark");
  }
};

const showGameWonModal = computed(
  () => store.getCorrectGuesses.length >= 2 && gameWonModalShown.value === false
);

const onOpenCorrectGuesses = () => {
  // without clearing timer, if user toggles correct guesses quickly, it will fade to background after timeout
  clearTimeout(timer);
  zindex.value = -1;
};

const onCloseCorrectGuesses = () => {
  timer = setTimeout(() => {
    zindex.value = 0;
  }, 2000);
};

onMounted(onToggleDarkMode);

store.startGame({ allAnswers });
// TODO: remove i18n
// TODO: extra not in spellingbee: track scores across days
// TODO: add shake animation on incorrect submission?
// https://www.reddit.com/r/webdev/comments/su6y4r/what_animations_are_used_in_wordle/
// need setTimeout to wait for animation before removing guess

const submitForm = () => {
  if (fullName.value.trim() !== "" && email.value.trim() !== "") {
    console.log("Full Name:", fullName.value);
    localStorage.setItem("full_name", fullName.value);
    console.log("Email:", email.value);
    localStorage.setItem("email", email.value);
    showInfo.value = true;
    popupModalShown.value = false; // Only close the modal if both fields are filled
  } else {
    // You can show a notification or a message to inform the user to fill out both fields
    console.warn("Both fields are required!");
  }
};
</script>

<template>
  <el-dialog
    v-model="showGameWonModal"
    @closed="gameWonModalShown = true"
    title="Congratulations!">
    <template #header="{ close, titleId, titleClass }">
      <div>
        <p style="font-size: 20px; font-weight: 600">
          <strong>Congratulations!</strong>
        </p>
      </div>
    </template>
    <GameWon />
  </el-dialog>
  <!-- Login modal is disabled, remove the v-if line to enable it. -->
  <el-dialog
    v-if="false"
    v-model="popupModalShown"
    title="Welcome to Spelling Bee!"
    align-center
    :width="20"
    :close-on-click-modal="false"
    :lock-scroll="true"
    :fullscreen="false"
    :show-close="false">
    <!-- Modal Header Start -->
    <template #header="{ close, titleId, titleClass }">
      <div class="my-header">
        <h4 :id="titleId" :class="titleClass">Welcome to Spelling Bee!</h4>
      </div>
    </template>
    <!-- Modal Header End -->

    <!-- Modal Body Start -->
    <form @submit.prevent="submitForm">
      <div>
        <p class="form-label" for="fullName">Full Name</p>
        <el-input
          v-model="fullName"
          id="fullName"
          ref="myInput"
          focus
          class="input-field"
          placeholder="Enter your full name"></el-input>
      </div>
      <div class="form-group">
        <p class="form-label" for="email">Email</p>
        <el-input
          v-model="email"
          id="email"
          placeholder="Enter your email"
          class="input-field"
          type="email"></el-input>
      </div>
      <div style="margin-top: 20px">
        <el-button
          type="primary"
          color="#fce303"
          native-type="submit"
          style="height: 2.5rem; margin-top: 1rem">
          <strong>Start Game</strong
          ><el-icon class="el-icon--right"><Aim /> </el-icon>
        </el-button>
      </div>
    </form>
    <!-- Modal Body End -->
  </el-dialog>
  <el-dialog v-model="showYesterdaysAnswers" :title="$t('Yesterdays Answers')">
    <YesterdaysAnswers />
  </el-dialog>
  <el-dialog v-model="showInfo" :title="$t('How to play')">
    <Info />
  </el-dialog>
  <div class="common-layout fireworks">
    <div class="beforeFireworks" v-if="showGameWonModal" />
    <div class="afterFireworks" v-if="showGameWonModal" />
    <el-header height="2em" id="title-header">
      <h2>
        <strong> Spelling Bee </strong>
      </h2>
    </el-header>
    <el-menu mode="horizontal" :ellipsis="false">
      <el-menu-item index="1" @click="showInfo = true">
        <el-tooltip :content="$t('Info')" placement="top">
          <el-icon class="menu-icon">
            <InfoFilled />
          </el-icon>
        </el-tooltip>
        <span class="responsive-menu-text">{{ $t("Info") }}</span>
      </el-menu-item>
      <!--
      <el-menu-item index="2" @click="showYesterdaysAnswers = true">
        <el-tooltip :content="$t('Yesterday')" placement="top">
          <el-icon class="menu-icon">
            <Calendar />
          </el-icon>
        </el-tooltip>
        <span class="responsive-menu-text">{{ $t("Yesterday") }}</span>
      </el-menu-item>
      -->
      <el-menu-item index="3">
        <el-switch
          v-model="darkmode"
          @change="onToggleDarkMode"
          class="darkmode-switch"
          style="--el-switch-on-color: $bl-yellow"
          inline-prompt
          size="large"
          :active-icon="Sunny"
          :inactive-icon="Moon" />
      </el-menu-item>
    </el-menu>
    <Progress />
    <CorrectGuesses
      @open="onOpenCorrectGuesses"
      @close="onCloseCorrectGuesses" />
    <Hive :ZIndex="zindex" />
  </div>
</template>

<style lang="scss">
@import "element-plus/dist/index.css";
@import "element-plus/theme-chalk/dark/css-vars.css";
@import "./assets/styles/fireworks.scss";
@import "./assets/styles/_variables.scss";

* {
  // stop double tap zoom on safari. often double tap keys in game.
  // TODO: register multiple click events when holding down button?
  // https://stackoverflow.com/a/53236027/6305204
  touch-action: manipulation;
  // https://stackoverflow.com/a/66103439/6305204
  -webkit-tap-highlight-color: transparent;
}

// https://element-plus.org/en-US/guide/theming.html#by-css-variable
:root {
  --el-color-success: $bl-yellow;
  --el-primary-color: $bl-yellow;
}

html {
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}

// don't allow words to be split. split on space between words
div {
  white-space: pre-wrap;
  word-break: break-word;
}

.darkmode-switch {
  margin-top: 5px;
}

h2 span {
  color: #bebebe;
  font-weight: lighter;
}

.main-container {
  padding-top: 3em;
}

.form-label {
  font-weight: bold;
  text-align: left;
}

.common-layout {
  max-width: 1000px;
  margin: auto;
}

.el-header h2 {
  padding: 0;
  margin: 0;
}
.el-menu--horizontal {
  border-top: solid 1px var(--el-menu-border-color);
  justify-content: space-between;
  .el-menu-item {
    padding: 0;
  }
  // yellow is too bright on light theme, use default blue
  // .el-menu-item.is-active {
  //   color: $bl-yellow !important;
  //   border-bottom-color: currentcolor;
  // }
}
.is-focused {
  border-color: $bl-yellow !important;
}
.is-selected {
  color: $bl-yellow !important;
  &::after {
    color: $bl-yellow;
    background-color: $bl-yellow !important;
  }
}

/* Default styles for both mobile and desktop */
.el-dialog {
  width: 30%; /* Default width for desktop */
  max-width: 80%; /* Max width for mobile */
  border-radius: 5px;
  box-shadow: 2px 2px 16px rgba(0, 0, 0, 0.2); /* Shadow effect */
}

/* Media query for mobile devices */
@media (max-width: 768px) {
  .el-dialog {
    width: 80%; /* Width for mobile */
    max-width: 80%; /* Max width for mobile */
  }
}
.el-table {
  --el-table-header-bg-color: unset;
}
.el-message--success {
  --el-message-bg-color: unset;
  --el-message-text-color: unset;
  background-color: $bl-yellow !important;
  color: black !important;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  // Available option for fonts:
  // font-family: Avenir, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 40px;
  padding: 0 10px;

  // account for 10px padding on either side of #app
  max-width: calc(100% - 20px);
  max-height: 100vh;
  #title-header {
    margin: 0;
    padding: 0;
    overflow: hidden;
  }
}

// pangram highlight used in multiple components
.pangram {
  font-weight: bold;
}

.toast-message {
  max-width: 80%;
  margin: 0, 1em;
  margin-top: 25vh;
}

html.dark {
  header strong {
    color: $bl-yellow;
  }
  .pangram {
    color: $bl-yellow;
  }
}

@media only screen and (max-height: 500px) {
  .toast-message {
    margin-top: 50px;
  }
}

@media only screen and (max-width: 700px) {
  #app {
    margin-top: 10px;
  }
  .menu-icon {
    margin: 19px 5px;
  }
}

@media only screen and (max-width: 400px) {
  .responsive-menu-text {
    display: none;
  }
}

form {
  div {
    margin-bottom: 10px;
  }

  label {
    display: block;
    margin-bottom: 5px;
  }
}
</style>
