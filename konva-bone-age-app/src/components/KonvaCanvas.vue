<template>
  <div id="konva-container-wrapper">
    <div id="konva-container"></div>
  </div>
</template>

<script setup>
import { onMounted, ref, watch, nextTick } from 'vue';
import Konva from 'konva';

// Props from App.vue
const props = defineProps({
  pointsData: Array,
  linesData: Array,
  currentLineType: String,
  currentLineColor: String,
  isDrawingLineMode: Boolean,
});

// Emits to App.vue
const emit = defineEmits(['updatePoints', 'updateLines', 'pointSelected']);

let stage = null;
let layer = null;
const selectedPointId = ref(null); // Konva ID of the highlighted point for canvas interaction
const lineStartPointId = ref(null); // Konva ID for the start of a line being drawn

const defaultPointStyle = { radius: 7, fill: 'blue', stroke: 'black', strokeWidth: 1 };
const highlightedPointStyle = { fill: 'red', stroke: 'darkred', strokeWidth: 2 };
// Note: lineStartPointHighlightStyle is handled directly in applyPointStyle

const calculateDistance = (p1, p2) => {
  if (!p1 || !p2) return 0;
  return Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2)).toFixed(1);
};

const findPointDataById = (pointId) => props.pointsData.find(p => p.id === pointId);

function applyPointStyle(konvaPoint, pointData, isHighlighted, isLineStart = false) {
  if (!konvaPoint || !pointData) return;
  let styleToApply = defaultPointStyle;
  if (isHighlighted) styleToApply = highlightedPointStyle;

  konvaPoint.fill(styleToApply.fill);
  konvaPoint.stroke(isLineStart ? 'green' : styleToApply.stroke);
  konvaPoint.strokeWidth(isLineStart ? highlightedPointStyle.strokeWidth + 1 : styleToApply.strokeWidth);
  konvaPoint.radius(styleToApply.radius);
}

function createKonvaPoint(pointData) {
  const konvaPoint = new Konva.Circle({
    id: pointData.id,
    x: pointData.x,
    y: pointData.y,
    draggable: true,
  });
  applyPointStyle(konvaPoint, pointData, false, false); // Apply default style initially

  konvaPoint.on('click tap', (evt) => {
    evt.cancelBubble = true;
    const currentId = konvaPoint.id();
    const currentPointData = findPointDataById(currentId);
    if (!currentPointData) return;

    if (props.isDrawingLineMode) {
      handleLineDrawingClick(currentId, currentPointData, konvaPoint);
    } else {
      handlePointSelectionClick(currentId, currentPointData, konvaPoint);
    }
  });

  konvaPoint.on('dragend', () => {
    const currentId = konvaPoint.id();
    const newPoints = props.pointsData.map(p =>
      p.id === currentId ? { ...p, x: konvaPoint.x(), y: konvaPoint.y() } : p
    );
    emit('updatePoints', newPoints);
  });

  konvaPoint.on('dragmove', () => {
    const currentId = konvaPoint.id();
    // Directly update line positions for real-time feedback
    props.linesData.forEach(line => {
      if (line.startPointId === currentId || line.endPointId === currentId) {
        const konvaLine = stage.findOne('#' + line.id);
        const distanceText = stage.findOne('#dist-text-' + line.id);
        if (konvaLine) {
          const startPtData = findPointDataById(line.startPointId);
          const endPtData = findPointDataById(line.endPointId);
          // Use current dragged position for one of the points
          const p1 = line.startPointId === currentId ? { x: konvaPoint.x(), y: konvaPoint.y() } : startPtData;
          const p2 = line.endPointId === currentId ? { x: konvaPoint.x(), y: konvaPoint.y() } : endPtData;

          if (p1 && p2) {
            konvaLine.points([p1.x, p1.y, p2.x, p2.y]);
            if (distanceText) {
              const dist = calculateDistance(p1, p2);
              distanceText.text(`${dist}px`);
              distanceText.x((p1.x + p2.x) / 2 + 5);
              distanceText.y((p1.y + p2.y) / 2 + 5);
            }
          }
        }
      }
    });
    layer.batchDraw();
  });
  return konvaPoint;
}

function handleLineDrawingClick(pointId, pointData, konvaPoint) {
  if (!lineStartPointId.value) {
    lineStartPointId.value = pointId;
    applyPointStyle(konvaPoint, pointData, false, true); // Mark as line start

    if (selectedPointId.value) { // Deselect any point selected in "select mode"
      const prevSelectedKonva = stage.findOne('#' + selectedPointId.value);
      if (prevSelectedKonva) applyPointStyle(prevSelectedKonva, findPointDataById(selectedPointId.value), false);
      selectedPointId.value = null;
      emit('pointSelected', null);
    }
  } else if (lineStartPointId.value !== pointId) {
    const newLineId = `line-${Date.now()}`;
    const newLinesArray = [
      ...props.linesData,
      {
        id: newLineId,
        startPointId: lineStartPointId.value,
        endPointId: pointId,
        type: props.currentLineType,
        color: props.currentLineColor,
      }
    ];
    emit('updateLines', newLinesArray);

    const startKonvaPoint = stage.findOne('#' + lineStartPointId.value);
    if (startKonvaPoint) applyPointStyle(startKonvaPoint, findPointDataById(lineStartPointId.value), false);
    lineStartPointId.value = null;
  }
  layer.draw();
}

function handlePointSelectionClick(pointId, pointData, konvaPoint) {
  if (selectedPointId.value && selectedPointId.value !== pointId) {
    const prevSelectedKonvaPoint = stage.findOne('#' + selectedPointId.value);
    if (prevSelectedKonvaPoint) applyPointStyle(prevSelectedKonvaPoint, findPointDataById(selectedPointId.value), false);
  }

  if (selectedPointId.value === pointId) {
    applyPointStyle(konvaPoint, pointData, false);
    selectedPointId.value = null;
    emit('pointSelected', null);
  } else {
    applyPointStyle(konvaPoint, pointData, true);
    selectedPointId.value = pointId;
    emit('pointSelected', { ...pointData });
  }
  layer.draw();
}

function synchronizeKonvaElements() {
  if (!stage || !layer) return;

  const currentPoints = props.pointsData || [];
  const currentLines = props.linesData || [];

  const existingKonvaPointIds = new Set(layer.find('Circle').map(c => c.id()));
  const propPointIds = new Set(currentPoints.map(p => p.id));

  existingKonvaPointIds.forEach(konvaId => {
    if (!propPointIds.has(konvaId)) stage.findOne('#' + konvaId)?.destroy();
  });

  currentPoints.forEach(pointData => {
    let konvaPoint = stage.findOne('#' + pointData.id);
    if (!konvaPoint) {
      konvaPoint = createKonvaPoint(pointData);
      layer.add(konvaPoint);
    } else {
      konvaPoint.x(pointData.x);
      konvaPoint.y(pointData.y);
    }
    const isSelected = selectedPointId.value === pointData.id && !props.isDrawingLineMode;
    const isLineStart = lineStartPointId.value === pointData.id && props.isDrawingLineMode;
    applyPointStyle(konvaPoint, pointData, isSelected, isLineStart);
  });

  const existingKonvaLineIds = new Set(layer.find('Line').map(l => l.id()));
  const propLineIds = new Set(currentLines.map(l => l.id));

  existingKonvaLineIds.forEach(konvaId => {
    if (!propLineIds.has(konvaId)) {
      stage.findOne('#' + konvaId)?.destroy();
      stage.findOne('#dist-text-' + konvaId)?.destroy();
    }
  });

  currentLines.forEach(lineData => {
    const startPoint = findPointDataById(lineData.startPointId);
    const endPoint = findPointDataById(lineData.endPointId);

    if (!startPoint || !endPoint) {
      stage.findOne('#' + lineData.id)?.destroy();
      stage.findOne('#dist-text-' + lineData.id)?.destroy();
      return;
    }

    let konvaLine = stage.findOne('#' + lineData.id);
    let distanceText = stage.findOne('#dist-text-' + lineData.id);

    if (!konvaLine) {
      konvaLine = new Konva.Line({ id: lineData.id, strokeWidth: 2 });
      layer.add(konvaLine);
      distanceText = new Konva.Text({ id: `dist-text-${lineData.id}`, fontSize: 12, fill: 'black' });
      layer.add(distanceText);
    }

    konvaLine.points([startPoint.x, startPoint.y, endPoint.x, endPoint.y]);
    konvaLine.stroke(lineData.color);
    konvaLine.dash(lineData.type === 'dashed' ? [10, 5] : []);
    // Future: konvaLine.tension(...) for curves

    const dist = calculateDistance(startPoint, endPoint);
    distanceText.text(`${dist}px`);
    distanceText.x((startPoint.x + endPoint.x) / 2 + 5);
    distanceText.y((startPoint.y + endPoint.y) / 2 + 5);
  });

  layer.batchDraw();
}

onMounted(() => {
  stage = new Konva.Stage({ container: 'konva-container', width: 800, height: 600 });
  layer = new Konva.Layer();
  stage.add(layer);

  synchronizeKonvaElements(); // Initial render

  stage.on('click tap', (e) => {
    if (e.target === stage) {
      if (props.isDrawingLineMode) {
        if (lineStartPointId.value) { // Cancel line drawing
          const startKonvaPoint = stage.findOne('#' + lineStartPointId.value);
          if (startKonvaPoint) applyPointStyle(startKonvaPoint, findPointDataById(lineStartPointId.value), false);
          lineStartPointId.value = null;
          layer.draw();
        } else { // Add new point
          const pos = stage.getPointerPosition();
          const pointId = `point-${Date.now()}`;
          const newPointData = {
            id: pointId, x: pos.x, y: pos.y,
            name: `P-${(props.pointsData?.length || 0) + 1}`,
            description: '', importance: 'medium', notes: ''
          };
          emit('updatePoints', [...(props.pointsData || []), newPointData]);
          // Optional: Auto-select for line drawing, after props update and Konva object is created
          nextTick(() => {
             const newKonvaPt = stage.findOne('#'+pointId);
             if(newKonvaPt) handleLineDrawingClick(pointId, newPointData, newKonvaPt);
          });
        }
      } else { // Selection mode
        if (selectedPointId.value) { // Deselect point
          const konvaPoint = stage.findOne('#' + selectedPointId.value);
          if (konvaPoint) applyPointStyle(konvaPoint, findPointDataById(selectedPointId.value), false);
          selectedPointId.value = null;
          emit('pointSelected', null);
          layer.draw();
        }
      }
    }
  });

  watch(() => [props.pointsData, props.linesData], () => {
    synchronizeKonvaElements();
  }, { deep: true });

  watch(() => props.isDrawingLineMode, () => {
    if (selectedPointId.value) {
      const konvaPoint = stage.findOne('#' + selectedPointId.value);
      if (konvaPoint) applyPointStyle(konvaPoint, findPointDataById(selectedPointId.value), false);
      selectedPointId.value = null;
      emit('pointSelected', null);
    }
    if (lineStartPointId.value) {
      const konvaPoint = stage.findOne('#' + lineStartPointId.value);
      if (konvaPoint) applyPointStyle(konvaPoint, findPointDataById(lineStartPointId.value), false);
      lineStartPointId.value = null;
    }
    layer.draw();
  });
});

</script>

<style scoped>
#konva-container-wrapper {
  display: inline-block;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
}
#konva-container {
  width: 800px;
  height: 600px;
  border: 1px solid #ccc;
  background-color: #fdfdfd;
}
</style>
