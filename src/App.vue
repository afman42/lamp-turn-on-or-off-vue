<script setup>
// No changes to the script section. The logic is perfect.
import { ref, computed, onUnmounted } from "vue";

const brightness = ref(50);
const isSliding = ref(false);
const sliderTrack = ref(null);

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

const containerStyle = computed(() => {
  if (brightness.value === 0) {
    return { backgroundColor: "#2c3e50", color: "#ffffff" };
  }
  return {
    backgroundColor: `rgba(247, 220, 111, ${brightness.value / 100})`,
    color: "#333333",
  };
});

const thumbStyle = computed(() => ({
  bottom: `${brightness.value}%`,
}));

onUnmounted(() => {
  handleSliderPointerEnd();
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
            <path d="M30 180H70V160H30V180Z" fill="#cccccc" />
            <path d="M45 160H55V90H45V160Z" fill="#cccccc" />
            <path
              d="M10 90H90L70 40H30L10 90Z"
              :fill="brightness > 0 ? '#555555' : '#444444'"
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
  </div>
</template>

<style scoped>
.lamp-container {
  height: inherit;
  display: flex;
  justify-content: center;
  align-items: center; /* Keep it vertically centered */

  font-family: Avenir, Helvetica, Arial, sans-serif;
  transition:
    background-color 0.4s ease,
    color 0.4s ease;
  overflow: hidden;
  box-sizing: border-box;

  /* Add padding so it doesn't touch the screen edge */
  padding: 0 5vw;
}

.main-content {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 40px;
}

.instructions {
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  text-align: center;
  padding: 0 1rem;
  box-sizing: border-box;
}
.instructions h1 {
  font-size: 2.5rem;
  font-weight: bold;
}
.instructions p {
  font-size: 1.2rem;
  opacity: 0.8;
}

.brightness-slider {
  display: flex;
  cursor: pointer;
  padding: 10px;
}

.slider-track {
  position: relative;
  top: 1.5rem;
  width: 12px;
  height: 200px;
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 6px;
}

.slider-thumb {
  position: absolute;
  left: 50%;
  transform: translate(-50%, 50%);
  width: 32px;
  height: 32px;
  background-color: white;
  border-radius: 50%;
  border: 3px solid #f0f0f0;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  transition: bottom 0.1s ease-out;
}
</style>
