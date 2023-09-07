<script setup lang="ts">
import { useMainStore } from "../store";
import { Share } from '@element-plus/icons-vue'

import axios from "axios";
import { ref } from 'vue';
import messages from "../../data/shareMessage.json";
const store = useMainStore();
let shareButton = ref("Share");
const shareScore = async () => {
  // Making a POST API call using Axios
  try {
    const randomNumber = Math.floor(Math.random() * 10);

    shareButton.value = `<svg viewBox="0 0 200 50" width="auto" height="auto">
                            <!-- Background -->
                            <rect x="0" y="0" width="200" height="50" fill="#fce303"></rect>
                            
                            <!-- Loader Circles -->
                            <circle id="dot1" cx="10" cy="25" r="10" fill="#4c51bf">
                              <animate attributeName="cx" dur="1.5s" repeatCount="indefinite" values="10; 190; 10"></animate>
                            </circle>
                            <circle id="dot2" cx="10" cy="25" r="10" fill="#4c51bf">
                              <animate attributeName="cx" dur="1.5s" repeatCount="indefinite" values="10; 190; 10" begin="0.3s"></animate>
                            </circle>
                            <circle id="dot3" cx="10" cy="25" r="10" fill="#4c51bf">
                              <animate attributeName="cx" dur="1.5s" repeatCount="indefinite" values="10; 190; 10" begin="0.6s"></animate>
                            </circle>
                            <circle id="dot4" cx="10" cy="25" r="10" fill="#4c51bf">
                              <animate attributeName="cx" dur="1.5s" repeatCount="indefinite" values="10; 190; 10" begin="0.9s"></animate>
                            </circle>
                          </svg>`;
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
    const score = store.getUserScore
    if (navigator.share) {
      navigator.share({
        title: messages[randomNumber].title,
        text: (messages[randomNumber].text).replace("<SCORE>", score.toString()),
        url: window.location.href,
      });
    } else {
      alert("Copied to clipboard!📋");
      const mytext = `${messages[randomNumber].title}\n${(messages[randomNumber].text).replace("<SCORE>", score.toString())}\n${window.location.href}`;
      await navigator.clipboard.writeText(mytext);
    }
  } catch (error) {
    console.error("Error sending data:", error);
  }

  shareButton.value = `Share`;
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
