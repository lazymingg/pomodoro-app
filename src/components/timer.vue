<script setup>
import { computed, ref, onMounted } from "vue";

onMounted(() => {
  if ("Notification" in window) {
    if (Notification.permission === "default") {
      Notification.requestPermission().then(permission => {
        console.log("Notification permission:", permission);
      });
    }
  } else {
    console.log("Notifications not supported in this browser.");
  }
});

function showNotification() {
  // Sử dụng đường dẫn tương đối để hoạt động cả local và GitHub Pages
  const audio = new Audio('./alarm.wav');
  
  // Xử lý lỗi nếu không load được audio
  audio.onerror = () => {
    console.warn("Could not load alarm sound");
  };

  const message = "⏰ Time's up! It's time to take a break or start the next session.";

  if ("Notification" in window) {
    if (Notification.permission === "granted") {
      new Notification("⏰ Time's up!", {
        body: "It's time to take a break or start the next session.",
        icon: './favicon.ico'
      });
      audio.play().catch(e => console.warn("Audio play failed:", e));
    } else if (Notification.permission === "default") {
      Notification.requestPermission().then(permission => {
        if (permission === "granted") {
          new Notification("⏰ Time's up!", {
            body: "It's time to take a break or start the next session.",
            icon: './favicon.ico'
          });
        } else {
          alert(message);
        }
        audio.play().catch(e => console.warn("Audio play failed:", e));
      });
    } else {
      // Permission denied
      alert(message);
      audio.play().catch(e => console.warn("Audio play failed:", e));
    }
  } else {
    // Notifications not supported
    alert(message);
    audio.play().catch(e => console.warn("Audio play failed:", e));
  }
}


defineEmits(["changeTimer"]);

const timeOptions = [
  { label: "Pomodoro", seconds: 25 * 60 * 1000 },
  { label: "Short Break", seconds: 5 * 60 * 1000 },
  { label: "Long Break", seconds: 15 * 60 * 1000 },
];

const currentOption = ref(timeOptions[0].label);
const timeLeft = ref(timeOptions[0].seconds);
let countdownInterval = null;

function formatTime(ms) {
  const minutes = Math.floor(ms / (60 * 1000));
  const secs = Math.round((ms % (60 * 1000)) / 1000);
  return `${String(minutes).padStart(2, "0")}:${String(secs).padStart(2, "0")}`;
}

const getTimeLeft = computed(() => formatTime(timeLeft.value));

function selectTimeOption(label) {
  const selected = timeOptions.find(option => option.label === label);
  if (selected && selected.label !== currentOption.value) {
    currentOption.value = selected.label;
    timeLeft.value = selected.seconds;
    if (countdownInterval) {
      clearInterval(countdownInterval);
      countdownInterval = null;
    }
  }
}

function count_down() {
  if (countdownInterval) clearInterval(countdownInterval);

  let start_time = Date.now();
  countdownInterval = setInterval(() => {
    const elapsed = Date.now() - start_time;
    timeLeft.value -= elapsed;
    start_time = Date.now();

    if (timeLeft.value <= 0) {
      timeLeft.value = 0;
      clearInterval(countdownInterval);
      countdownInterval = null;
      showNotification();
      selectTimeOption(currentOption.value);
      timeLeft.value = timeOptions.find(option => option.label === currentOption.value).seconds;
    }
  }, 250);
}
</script>

<template>
  <div class="timer-box">
    <div class="time-bar">
      <button v-for="option in timeOptions" :key="option.label"
        @click="selectTimeOption(option.label); $emit('changeTimer', option.label)"
        :class="{ active: currentOption === option.label }">
        {{ option.label }}
      </button>
    </div>
    <div class="remaining-time">
      {{ getTimeLeft }}
    </div>
    <button class="start-btn" @click="count_down()">Start</button>
  </div>
</template>

<style scoped>
.timer-box {
  width: 35rem;
  background-color: var(--color-background-soft);
  margin-bottom: 5rem;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.time-bar {
  margin-top: 1rem;
  display: flex;
  justify-content: center;
}

.time-bar button {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  border-radius: 5px;
  background-color: transparent;
  width: 7rem;
  height: 2.5rem;
  font-weight: 600;
  color: var(--color-heading);
  border: none;
  cursor: pointer;
  margin: 0 0.5rem;
  transition: background-color 0.3s ease;
  text-align: center;
  line-height: 2.5rem;
}

.time-bar button:hover {
  color: var(--color-text);
  background-color: var(--color-background-mute);
}

.time-bar button.active {
  background-color: var(--color-border);
  font-weight: 700;
}

.remaining-time {
  color: var(--color-text);
  width: 70%;
  height: 50%;
  text-align: center;
  font-size: 8rem;
  font-weight: 500;
}

.start-btn {
  background-color: var(--color-background-mute);
  color: var(--color-heading);
  border: none;
  border-radius: 8px;
  padding: 0.8rem 2rem;
  font-size: 1.2rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 1.5rem;
  margin-bottom: 1.5rem;
}

.start-btn:hover {
  background-color: var(--color-border-hover);
}
</style>