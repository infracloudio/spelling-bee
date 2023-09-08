<script setup lang="ts">
import { useMainStore } from "../store";
import { Share } from '@element-plus/icons-vue'

import axios from "axios";

const store = useMainStore();
const shareScore = async () => {
  // Making a POST API call using Axios
  try {
    const response = await axios.post(
      import.meta.env.VITE_GSA_URL || "",
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
    );
    console.log("Data sent successfully:", response.data);
    // Example functionality for the share button
    if (navigator.share) {
      navigator.share({
        title: "My Spelling Bee Score!",
        text: `I scored ${store.getUserScore} on Spelling Bee! Can you beat my score?`,
        url: window.location.href,
      });
    } else {
      alert("Copied to clipboard!📋");
      const mytext = `My Spelling Bee Score!\nI scored ${store.getUserScore} on Spelling Bee! Can you beat my score?\n${window.location.href}`;
      await navigator.clipboard.writeText(mytext);
    }
  } catch (error) {
    console.error("Error sending data:", error);
  }
};
</script>

<template>
  <div>
     <p>
     <h2>Your Score: {{ store.getUserScore }} 🎉</h2>
     <p>Well done! You're doing great.</p>
     <p>🚀 Share the results with your friends on Twitter/LinkedIn and win some cool swags at the InfraCloud booth! 🎁🌟
     </p>
     <p>
        And stand a chance to win an electric
        <strong>hoverboard! 💨</strong>, do keep an 👁️ on your social media platform! 😉🏆
     <p>
        <img height="90" src="../assets/hoverboard-400.png" alt="Mega-prize" class="hoverboard-icon" />
     </p>
     <el-button @click="shareScore" style="height: 3rem; margin-top: 1rem; width: 75%; font-size: 18px;">
        <el-icon
           class="el-icon--left">
           <Share />
        </el-icon>
        <strong>Share</strong>
     </el-button>
     </p>
     </p>
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
