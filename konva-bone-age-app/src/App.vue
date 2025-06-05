<template>
  <header>
    <h1>Bone Age Analysis Tool</h1>
  </header>
  <main>
    <KonvaCanvas
      :pointsData="points"
      :linesData="lines"
      :currentLineType="currentLineType"
      :currentLineColor="currentLineColor"
      :isDrawingLineMode="isDrawingLineMode"
      @pointSelected="handlePointSelected"
      @updatePoints="handleUpdatePoints"
      @updateLines="handleUpdateLines"
    />
    <div class="side-panels">
      <ControlPanel
        :currentLineType="currentLineType"
        :currentLineColor="currentLineColor"
        :isDrawingLineMode="isDrawingLineMode"
        @update:lineType="val => currentLineType = val"
        @update:lineColor="val => currentLineColor = val"
        @update:drawingMode="val => isDrawingLineMode = val"
      />
      <InfoPanel
        :selectedPointData="selectedPointDetails"
        @update:pointData="handlePointDataUpdate"
      />
    </div>
  </main>
  <footer>
    <p>Mode: <span class="mode-text">{{ isDrawingLineMode ? 'Draw Lines' : 'Select Points' }}</span> | Type: <span class="type-text">{{ currentLineType }}</span> | Color: <span :style="{ color: currentLineColor, fontWeight: 'bold' }">{{ currentLineColor }}</span></p>
  </footer>
</template>

<script setup>
import { ref } from 'vue';
import KonvaCanvas from './components/KonvaCanvas.vue';
import ControlPanel from './components/ControlPanel.vue';
import InfoPanel from './components/InfoPanel.vue';

// Master data - lifted from KonvaCanvas
const points = ref([]);
const lines = ref([]);

const currentLineType = ref('straight');
const currentLineColor = ref('blue');
const isDrawingLineMode = ref(true);
const selectedPointDetails = ref(null);

const handlePointSelected = (pointData) => {
  if (pointData) {
    selectedPointDetails.value = { ...pointData };
  } else {
    selectedPointDetails.value = null;
  }
};

const handlePointDataUpdate = (updatedPoint) => {
  const index = points.value.findIndex(p => p.id === updatedPoint.id);
  if (index !== -1) {
    points.value[index] = { ...points.value[index], ...updatedPoint };
    if (selectedPointDetails.value && selectedPointDetails.value.id === updatedPoint.id) {
      selectedPointDetails.value = { ...points.value[index] };
    }
  }
};

// Handlers for updates from KonvaCanvas
const handleUpdatePoints = (newPoints) => {
  points.value = newPoints;
};

const handleUpdateLines = (newLines) => {
  lines.value = newLines;
};

</script>

<style>
/* Basic global styles */
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  margin: 0;
  color: #333;
  background-color: #f4f4f8;
}

header {
  padding: 15px 20px;
  background-color: #4A90E2;
  color: white;
  border-bottom: 2px solid #357ABD;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

header h1 {
  margin: 0;
  font-size: 1.8em;
}

main {
  display: flex;
  flex-wrap: nowrap; /* Prevent wrapping to keep side panel next to canvas */
  padding: 20px;
  justify-content: flex-start;
  gap: 20px;
  align-items: flex-start; /* Align items to the top */
  height: calc(100vh - 150px); /* Adjust based on header/footer height */
  overflow-x: auto; /* Allow horizontal scrolling if content overflows */
}

.side-panels {
  display: flex;
  flex-direction: column;
  gap: 20px;
  min-width: 270px; /* Ensure side panels have enough width */
  max-width: 320px; /* And not too much */
}

footer {
  padding: 12px 20px;
  background-color: #333;
  color: #f0f0f0;
  border-top: 2px solid #555;
  text-align: center;
  font-size: 0.9em;
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  box-sizing: border-box;
  box-shadow: 0 -2px 4px rgba(0,0,0,0.1);
  z-index: 1000; /* Ensure footer is on top */
}

footer .mode-text, footer .type-text {
  font-weight: bold;
  margin: 0 3px;
}

/* Ensure KonvaCanvas has a defined size or flex properties */
/* KonvaCanvas.vue already has a width/height for its container */
/* ControlPanel.vue and InfoPanel.vue have min-width */
</style>
