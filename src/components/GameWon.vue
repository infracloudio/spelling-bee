<script setup lang="ts">
import { useMainStore } from "../store";
import { Share } from "@element-plus/icons-vue";

import axios from "axios";
import { ref } from "vue";
import messages from "../../data/shareMessage.json";
import { ElMessage } from "element-plus";

const store = useMainStore();
const loading = ref(false);

const attemptShare = (shareData: object) => {
  return navigator.canShare && navigator.canShare(shareData) && navigator.share;
};

// TODO: remove this?
const handleShareFailure = () => {
  store.showMessage({
    type: "warning",
    message: "Error submitting score, try again!",
  });
};

const shareScore = async () => {
  // We don't wait here as it causes the clipboard to fail on Firefox.
  axios
    .post(
      import.meta.env.VITE_GSA_URL,
      {
        Score: store.getUserScore,
        Name: localStorage.getItem("full_name"),
        Email: localStorage.getItem("email"),
      },
      {
        headers: {
          "Content-Type": "text/plain;charset=utf-8",
        },
      }
    )
    .then((response) => {
      console.log("Data sent successfully:", response.data);
    })
    .catch((error) => {
      console.error("Error sending data:", error);
    });

  const textToShare = messages[
    Math.floor(Math.random() * messages.length)
  ].replace("<SCORE>", store.getUserScore);

  // Based on
  // https://github.com/cwackerfuss/react-wordle/blob/adfa447/src/lib/share.ts
  const shareData = { text: textToShare };
  let shareSuccess = false;

  try {
    if (attemptShare(shareData)) {
      navigator.share(shareData);
      shareSuccess = true;
    }
  } catch (error) {
    shareSuccess = false;
  }

  try {
    if (!shareSuccess) {
      if (navigator.clipboard) {
        navigator.clipboard.writeText(textToShare);
        store.showMessage({
          type: "success",
          message: "Copied to clipboard! 📋",
        });
      } else {
        handleShareFailure();
      }
    }
  } catch (error) {
    handleShareFailure();
  }
};
</script>

<template>
  <div>
    <h2>Your Score: {{ store.getUserScore }} 🎉</h2>
    <p>Well done! You're doing great.</p>
    <p>
      🚀 Share the results with your friends on Twitter/LinkedIn and win some
      cool swags at the InfraCloud booth! 🎁🌟
    </p>
    <p>
      And stand a chance to win an electric
      <strong>hoverboard! 💨</strong>, do keep an 👁️ on your social media
      platform! 😉🏆
    </p>
    <p>
      <img
        height="90"
        src="../assets/hoverboard-200.png"
        alt="Mega-prize"
        class="hoverboard-icon" />
    </p>
    <el-button
      v-loading="loading"
      @click="shareScore"
      style="height: 3rem; margin-top: 1rem; width: 75%; font-size: 18px">
      <el-icon class="el-icon--left">
        <Share />
      </el-icon>
      <strong>Share</strong>
    </el-button>
  </div>
</template>

<style scoped>
button {
  background-color: #fce303;
  box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.2);
  color: #303133;
  padding: 12px 24px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  cursor: pointer;
  border-radius: 12px;
  transition: background-color 0.2s, transform 0.2s;
}

button:hover {
  background-color: #fce303;
  color: #303133;
}

button:active {
  background-color: #d2c800;
  transform: scale(0.95);
  transition-duration: 0.1s;
}
</style>
