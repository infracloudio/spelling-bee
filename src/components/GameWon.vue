<script setup lang="ts">
import { useMainStore } from "../store";
import axios from "axios";
import { ref } from "vue";
import messages from "../../data/shareMessage.json";
const store = useMainStore();
let shareButton = ref("Share");

const attemptShare = (shareData: object) => {
  return (
    // Deliberately exclude Firefox Mobile, because its Web Share API isn't working correctly
    // browser.name?.toUpperCase().indexOf('FIREFOX') === -1 &&
    // webShareApiDeviceTypes.indexOf(device.type ?? '') !== -1 &&
    navigator.canShare && navigator.canShare(shareData) && navigator.share
  );
};

const shareScore = async () => {
  const randomNumber = Math.floor(Math.random() * 10);
  const score = store.getUserScore;

  const mytext = `${messages[randomNumber].title}\n${messages[
    randomNumber
  ].text.replace("<SCORE>", score.toString())}\n${window.location.href}`;

  if (navigator.clipboard) {
    navigator.clipboard.writeText(mytext);
  }

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

    const shareObject = {
      title: messages[randomNumber].title,
      text: messages[randomNumber].text.replace("<SCORE>", score.toString()),
      url: window.location.href,
    };
    if (attemptShare(shareObject)) {
      navigator.share(shareObject);
    } else {
      if (navigator.clipboard) {
        alert(
          "Oops something is not right, we have copied the message to your clipboard :D"
        );
      } else {
        alert("Hmm, looks like we can't share! :(");
      }
    }
  } catch (error) {
    console.error("Error sending data:", error);
  }

  shareButton.value = `Share`;
};
</script>

<template>
  <div>
    <h2>Your Score: {{ store.getUserScore }} 🎉</h2>
    <p>Well done! You did great. Share your score with your friends!</p>
    <p>
      Share your best streak with friends on Twitter & invite them for a game of
      cloud native world
    </p>
    <p>
      Help us add more words:
      <a
        href="https://github.com/infracloudio/spelling-bee"
        target="_blank"
        rel="noopener noreferrer"
        >infracloudio/spelling-bee</a
      >
    </p>
    <button @click="shareScore" v-html="shareButton"></button>
  </div>
</template>

<style scoped>
button {
  width: 6rem;
  height: 4rem;
  background-color: rgb(252, 227, 3);
  /* Green background */
  border: none;
  /* No border */
  color: black;
  /* White text */
  padding: 12px 24px;
  /* Some padding */
  text-align: center;
  /* Centered text */
  text-decoration: none;
  /* No underline */
  display: inline-block;
  /* Display as inline-block */
  font-size: 16px;
  /* Change default font size */
  margin: 4px 2px;
  /* Some margin */
  cursor: pointer;
  /* Pointer/hand icon */
  border-radius: 12px;
  /* Rounded corners */
}

button:hover {
  background-color: #fce303;
  /* Darker green variant on mouse-over */
}
</style>
