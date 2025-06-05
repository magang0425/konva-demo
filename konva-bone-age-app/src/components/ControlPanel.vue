<template>
  <div class="control-panel">
    <h4>Controls</h4>
    <div>
      <label for="line-type-select">Line Type: </label>
      <select id="line-type-select" :value="currentLineType" @change="onLineTypeChange">
        <option value="straight">Straight</option>
        <option value="dashed">Dashed</option>
        <option value="curve">Curve (Bezier)</option>
        <option value="smooth">Smooth Curve</option>
      </select>
    </div>
    <div style="margin-top: 10px;">
      <label>Line Color: </label>
      <button v-for="color in colors" :key="color"
              @click="onColorChange(color)"
              :class="{ 'selected-color': color === currentLineColor }"
              :style="{ backgroundColor: color, marginRight: '5px', padding: '5px 10px', border: '1px solid #000', color: getButtonTextColor(color) }">
        {{ color }}
      </button>
    </div>
     <div style="margin-top: 10px;">
      <label for="drawing-mode-toggle">Drawing Mode: </label>
      <button @click="toggleDrawingMode">
        {{ isDrawingLineMode ? 'Draw Lines' : 'Select Points' }}
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'; // ref is not strictly needed here if all state is via props

// Props to receive current selections from parent (App.vue)
const props = defineProps({
  currentLineType: String,
  currentLineColor: String,
  isDrawingLineMode: Boolean,
});

// Emits to send changes to parent (App.vue)
const emit = defineEmits(['update:lineType', 'update:lineColor', 'update:drawingMode']);

const colors = ['blue', 'red', 'green', 'yellow', 'black', 'purple']; // Available colors

const onLineTypeChange = (event) => {
  emit('update:lineType', event.target.value);
};

const onColorChange = (color) => {
  emit('update:lineColor', color);
};

const toggleDrawingMode = () => {
  emit('update:drawingMode', !props.isDrawingLineMode);
}

// Helper to determine button text color for better contrast
const getButtonTextColor = (backgroundColor) => {
  // Simple contrast logic: light background -> black text, dark background -> white text
  const darkColors = ['blue', 'red', 'green', 'black', 'purple']; // Add more dark colors if needed
  if (darkColors.includes(backgroundColor.toLowerCase())) {
    return 'white';
  }
  return 'black';
};
</script>

<style scoped>
.control-panel {
  padding: 15px;
  border: 1px solid #ccc;
  margin-left: 20px; /* If App.vue uses flex display */
  min-width: 220px; /* Adjusted min-width */
  background-color: #f9f9f9;
}
.control-panel div {
  margin-bottom: 12px;
}
label {
  margin-right: 8px;
  font-weight: bold;
}
.selected-color {
  border: 3px solid navy !important;
  box-shadow: 0 0 8px rgba(0,0,100,0.6);
  transform: scale(1.1); /* Slightly larger when selected */
}
button, select {
  cursor: pointer;
  padding: 6px 10px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
select {
    width: calc(100% - 22px); /* Adjust width for select box */
}
button:hover {
  background-color: #e0e0e0;
}
.selected-color:hover {
    /* Maintain selected look on hover */
    border: 3px solid navy !important;
}
</style>
