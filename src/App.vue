<script setup>
import { ref, computed, onUnmounted } from "vue";

// STATE
const isLampOn = ref(false); // Holds the lamp's on/off state.
const startY = ref(0); // Holds the starting Y position for both touch and mouse.
const isDragging = ref(false); // Tracks if the mouse button is currently held down.

// --- TOUCH EVENT HANDLERS (for Mobile & Tablet) ---
function handleTouchStart(event) {
  startY.value = event.touches[0].clientY;
}

function handleTouchEnd(event) {
  const endY = event.changedTouches[0].clientY;
  const deltaY = startY.value - endY; // Positive for UP, negative for DOWN

  // Apply the logic
  applySwipeLogic(deltaY);
}

// --- MOUSE EVENT HANDLERS (for Desktop) ---
function handleMouseDown(event) {
  // We only care about the primary (left) mouse button
  if (event.button !== 0) return;

  isDragging.value = true;
  startY.value = event.clientY;

  // We add listeners to the whole window. This lets the user
  // drag outside the component and still have it register.
  window.addEventListener("mousemove", handleMouseMove);
  window.addEventListener("mouseup", handleMouseUp);
}

function handleMouseMove(event) {
  // Prevents annoying text selection while dragging
  event.preventDefault();
}

function handleMouseUp(event) {
  if (!isDragging.value) return;

  isDragging.value = false;
  const endY = event.clientY;
  const deltaY = startY.value - endY; // Positive for UP, negative for DOWN

  // Apply the same logic as the touch events
  applySwipeLogic(deltaY);

  // Clean up the window listeners to prevent memory leaks
  window.removeEventListener("mousemove", handleMouseMove);
  window.removeEventListener("mouseup", handleMouseUp);
}

// --- UNIFIED LOGIC ---
// This function contains the shared logic for turning the lamp on/off.
function applySwipeLogic(deltaY) {
  const swipeThreshold = 50; // 50 pixels

  if (deltaY > swipeThreshold) {
    // If swiped/dragged UP, turn the lamp ON.
    isLampOn.value = true;
  } else if (deltaY < -swipeThreshold) {
    // If swiped/dragged DOWN, turn the lamp OFF.
    isLampOn.value = false;
  }
}

// --- COMPUTED PROPERTY for STYLING ---
const containerStyle = computed(() => ({
  backgroundColor: isLampOn.value ? "#f7dc6f" : "#2c3e50",
  color: isLampOn.value ? "#333333" : "#ffffff",
  // Change cursor to show the element is grabbable
  cursor: isDragging.value ? "grabbing" : "grab",
}));

// --- LIFECYCLE HOOK for CLEANUP ---
// This ensures that if the component is ever removed from the page
// while a drag is in progress, we don't leave orphaned event listeners.
onUnmounted(() => {
  window.removeEventListener("mousemove", handleMouseMove);
  window.removeEventListener("mouseup", handleMouseUp);
});
</script>

<template>
  <div
    class="lamp-container"
    :style="containerStyle"
    @touchstart.prevent="handleTouchStart"
    @touchend="handleTouchEnd"
    @mousedown="handleMouseDown"
  >
    <div class="lamp-visual">
      <svg
        v-if="!isLampOn"
        width="100"
        height="180"
        viewBox="0 0 100 180"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path d="M30 180H70V160H30V180Z" fill="#cccccc" />
        <path d="M45 160H55V90H45V160Z" fill="#cccccc" />
        <path d="M10 90H90L70 40H30L10 90Z" fill="#555555" />
      </svg>

      <svg
        v-else
        width="100"
        height="180"
        viewBox="0 0 100 180"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
      >
        <defs>
          <radialGradient id="glow" cx="0.5" cy="0.5" r="0.5">
            <stop offset="0%" stop-color="#ffffcc" />
            <stop offset="100%" stop-color="rgba(255, 255, 204, 0)" />
          </radialGradient>
        </defs>
        <rect x="0" y="0" width="100" height="120" fill="url(#glow)" />
        <path d="M30 180H70V160H30V180Z" fill="#e0e0e0" />
        <path d="M45 160H55V90H45V160Z" fill="#e0e0e0" />
        <path d="M10 90H90L70 40H30L10 90Z" fill="#ffd700" />
        <circle cx="50" cy="65" r="20" fill="#ffffff" />
      </svg>
    </div>

    <div class="instructions">
      <h1 v-if="!isLampOn">Lamp is Off</h1>
      <h1 v-else>Lamp is On</h1>
      <p>Swipe or Drag Up to Turn On</p>
      <p>Swipe or Drag Down to Turn Off</p>
    </div>
  </div>
</template>

<style scoped>
.lamp-container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  transition:
    background-color 0.6s ease,
    color 0.6s ease;

  /* touch-action: none; is still useful for touch devices */
  touch-action: none;
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
  user-select: none; /* Prevents text selection during drag */
}

/* Rest of the styles are the same */
.lamp-visual {
  margin-bottom: 2rem;
}
.instructions h1 {
  font-size: 2.5rem;
  font-weight: bold;
}
.instructions p {
  font-size: 1.2rem;
  opacity: 0.8;
}
</style>
