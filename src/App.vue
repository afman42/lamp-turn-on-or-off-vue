<script setup>
import { ref, computed, onMounted, onUnmounted, watchEffect } from "vue";
import { Capacitor } from "@capacitor/core";
import { App } from "@capacitor/app";
import { ScreenBrightness } from "@capacitor-community/screen-brightness";

// --- STATE MANAGEMENT ---
const brightness = ref(50);
const isSliding = ref(false);
const sliderTrack = ref(null);
const platform = ref("web");
const theme = ref("light");

// --- SLIDER LOGIC ---
function updateBrightnessFromEvent(event) {
  if (!sliderTrack.value) return;
  const trackRect = sliderTrack.value.getBoundingClientRect();
  const clientY = event.touches ? event.touches[0].clientY : event.clientY;
  const yRelativeToTrack = clientY - trackRect.top;
  const percentage = 1 - yRelativeToTrack / trackRect.height;
  const newBrightness = Math.round(percentage * 100);
  brightness.value = Math.max(0, Math.min(100, newBrightness));
}

function handleSliderPointerStart(event) {
  isSliding.value = true;
  updateBrightnessFromEvent(event);
  window.addEventListener("mousemove", handleSliderPointerMove);
  window.addEventListener("touchmove", handleSliderPointerMove);
  window.addEventListener("mouseup", handleSliderPointerEnd);
  window.addEventListener("touchend", handleSliderPointerEnd);
}

function handleSliderPointerMove(event) {
  if (!isSliding.value) return;
  event.preventDefault();
  updateBrightnessFromEvent(event);
}

function handleSliderPointerEnd() {
  isSliding.value = false;
  window.removeEventListener("mousemove", handleSliderPointerMove);
  window.removeEventListener("touchmove", handleSliderPointerMove);
  window.removeEventListener("mouseup", handleSliderPointerEnd);
  window.removeEventListener("touchend", handleSliderPointerEnd);
}

// --- NATIVE THEME & PLATFORM LOGIC ---
const darkModeQuery = window.matchMedia("(prefers-color-scheme: dark)");

const updateTheme = () => {
  const isDark = darkModeQuery.matches;
  theme.value = isDark ? "dark" : "light";
  document.documentElement.setAttribute("data-theme", theme.value);
};

let appStateListener = null;

onMounted(async () => {
  // 1. Check the platform on startup.
  platform.value = Capacitor.getPlatform();

  // 2. Set the initial theme.
  updateTheme();

  // 3. Listen for live theme changes from the OS.
  darkModeQuery.addEventListener("change", updateTheme);

  // 4. Listen for when the app is resumed to re-check the theme.
  if (platform.value !== "web") {
    appStateListener = App.addListener("appStateChange", ({ isActive }) => {
      if (isActive) {
        updateTheme();
      }
    });
  }
});

watchEffect(async () => {
  if (platform.value !== "web") {
    let brightnessAndroid = brightness.value / 100;
    await ScreenBrightness.setBrightness({ brightness: brightnessAndroid });
  }
});

onUnmounted(() => {
  // 5. Clean up all listeners to prevent memory leaks.
  darkModeQuery.removeEventListener("change", updateTheme);
  if (appStateListener) {
    appStateListener.remove();
  }
  handleSliderPointerEnd();
});

// --- COMPUTED PROPERTIES ---
const thumbStyle = computed(() => ({
  bottom: `${brightness.value}%`,
}));

const containerStyle = computed(() => {
  if (brightness.value === 0) {
    return { backgroundColor: "#2c3e50", color: "#ffffff" };
  }
  return {
    backgroundColor: `rgba(247, 220, 111, ${brightness.value / 100})`,
    color: "#333333",
  };
});
</script>

<template>
  <div class="lamp-container" :style="containerStyle">
    <div class="main-content">
      <div class="lamp-visual">
        <svg
          width="150"
          height="270"
          viewBox="0 0 100 180"
          xmlns="http://www.w3.org/2000/svg"
        >
          <g :style="{ opacity: 0.5 + brightness / 200 }">
            <path class="lamp-base" d="M30 180H70V160H30V180Z" />
            <path class="lamp-base" d="M45 160H55V90H45V160Z" />
            <path
              class="lamp-shade"
              :class="{ 'lamp-shade-off': brightness === 0 }"
              d="M10 90H90L70 40H30L10 90Z"
            />
          </g>
          <circle
            v-if="brightness > 0"
            cx="50"
            cy="65"
            r="25"
            fill="#ffffff"
            :fill-opacity="brightness / 100"
          />
        </svg>
      </div>
      <div
        class="brightness-slider"
        ref="sliderTrack"
        @mousedown.prevent="handleSliderPointerStart"
        @touchstart.prevent="handleSliderPointerStart"
      >
        <div class="slider-track">
          <div class="slider-thumb" :style="thumbStyle"></div>
        </div>
      </div>
    </div>

    <div class="instructions">
      <h1>Brightness: {{ brightness }}%</h1>
      <p>Click or drag the slider to adjust</p>
    </div>

    <div class="status-panel">
      <div class="status-title">Platform Status</div>
      <div class="status-item">
        <span class="status-icon">
          <svg
            v-if="platform === 'android'"
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M17 18a5 5 0 0 0-10 0"></path>
            <line x1="12" y1="2" x2="12" y2="8"></line>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
            <line x1="18.36" y1="4.22" x2="19.78" y2="5.64"></line>
          </svg>
          <svg
            v-else
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <circle cx="12" cy="12" r="10"></circle>
            <line x1="2" y1="12" x2="22" y2="12"></line>
            <path
              d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"
            ></path>
          </svg>
        </span>
        <span v-if="platform === 'android'">Native Android App</span>
        <span v-else>Standard Website</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* --- 1. THEME VARIABLES --- */
:root,
[data-theme="light"] {
  --background-color: #f0f0f0;
  --text-color: #1a1a1a;
  --text-color-light: #555555;
  --slider-thumb-color: #ffffff;
  --panel-bg-color: rgba(255, 255, 255, 0.5);
  --panel-border-color: rgba(0, 0, 0, 0.1);
  --icon-color: #333333;
  --lamp-base-color: #cccccc;
  --lamp-shade-color: #555555;
  --lamp-glow-color: #ffffff;
}

[data-theme="dark"] {
  --background-color: #121212;
  --text-color: #f0f0f0;
  --text-color-light: #bbbbbb;
  --slider-thumb-color: #444444;
  --panel-bg-color: rgba(0, 0, 0, 0.4);
  --panel-border-color: rgba(255, 255, 255, 0.2);
  --icon-color: #eeeeee;
  --lamp-base-color: #333333;
  --lamp-shade-color: #666666;
  --lamp-glow-color: #ffffdd;
}

/* --- 2. LAYOUT & GLOBAL STYLES --- */
.lamp-container {
  height: inherit;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  box-sizing: border-box;
  padding: 0 5vw;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  background-color: var(--background-color);
  color: var(--text-color);
  transition:
    background-color 0.4s ease,
    color 0.4s ease;
}

.main-content {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 40px;
}
.instructions {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
}
.instructions h1 {
  font-weight: bold;
  font-size: 2.5rem;
}
.instructions p {
  opacity: 0.8;
  font-size: 1.2rem;
  color: var(--text-color-light);
}

/* --- 3. COMPONENT & STATUS PANEL STYLES --- */
.status-panel {
  position: absolute;
  top: 20px;
  left: 20px;
  padding: 12px 16px;
  border-radius: 12px;
  background-color: var(--panel-bg-color);
  border: 1px solid var(--panel-border-color);
  backdrop-filter: blur(5px);
  font-size: 0.9rem;
}
.status-title {
  font-weight: bold;
  margin-bottom: 8px;
  opacity: 0.7;
  font-size: 0.8rem;
}
.status-item {
  display: flex;
  align-items: center;
  gap: 8px;
}
.status-icon svg {
  stroke: var(--icon-color);
}

.lamp-base {
  fill: var(--lamp-base-color);
}
.lamp-shade {
  fill: var(--lamp-shade-color);
}
.lamp-shade-off {
  fill: var(--lamp-base-color);
}
.lamp-glow {
  fill: var(--lamp-glow-color);
}

.brightness-slider {
  display: flex;
  cursor: pointer;
  padding: 10px;
}
.slider-track {
  position: relative;
  width: 12px;
  top: 1.5rem;
  height: 200px;
  background-color: white;
  border-radius: 6px;
}
.slider-thumb {
  position: absolute;
  left: 50%;
  transform: translate(-50%, 50%);
  width: 32px;
  height: 32px;
  border-radius: 50%;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  background-color: white;
  border: 3px solid var(--slider-thumb-color);
  transition: bottom 0.1s ease-out;
}

@media screen and (max-width: 768px) {
  .status-panel {
    top: 5rem;
  }
}
</style>
