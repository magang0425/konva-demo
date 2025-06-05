<template>
  <div class="info-panel" v-if="selectedPointData">
    <h4>Point Details</h4>
    <div>
      <label for="point-name">Name:</label>
      <input type="text" id="point-name" :value="editablePoint.name" @input="updateField('name', $event.target.value)">
    </div>
    <div>
      <label for="point-desc">Description:</label>
      <textarea id="point-desc" :value="editablePoint.description" @input="updateField('description', $event.target.value)"></textarea>
    </div>
    <div>
      <label for="point-importance">Importance:</label>
      <select id="point-importance" :value="editablePoint.importance" @input="updateField('importance', $event.target.value)">
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
      </select>
    </div>
    <div>
      <label for="point-notes">Notes:</label>
      <textarea id="point-notes" :value="editablePoint.notes" @input="updateField('notes', $event.target.value)"></textarea>
    </div>
    <!-- <button @click="saveChanges">Save Changes</button> -->
    <!-- Auto-save on input for now, explicit save can be added if needed -->
  </div>
  <div class="info-panel placeholder" v-else>
    <p>Select a point to see its details.</p>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  selectedPointData: Object // This will be the full point object from App.vue
});

const emit = defineEmits(['update:pointData']);

// Local editable copy of the point data to avoid directly mutating props
const editablePoint = ref({ name: '', description: '', importance: 'medium', notes: '' });

watch(() => props.selectedPointData, (newVal) => {
  if (newVal) {
    // Create a fresh copy when the selected point changes
    // Ensure all expected fields are present, defaulting if necessary
    editablePoint.value = {
      name: newVal.name || '',
      description: newVal.description || '',
      importance: newVal.importance || 'medium',
      notes: newVal.notes || ''
    };
  } else {
    // Reset if no point is selected
    editablePoint.value = { name: '', description: '', importance: 'medium', notes: '' };
  }
}, { deep: true, immediate: true });

const updateField = (field, value) => {
  if (props.selectedPointData) { // Ensure a point is actually selected
    editablePoint.value[field] = value;
    // Emit the entire updated point object, including its ID
    // It's crucial that props.selectedPointData.id is valid here.
    emit('update:pointData', { ...editablePoint.value, id: props.selectedPointData.id });
  }
};

</script>

<style scoped>
.info-panel {
  padding: 15px;
  border: 1px solid #ccc;
  /* margin-left: 20px; */ /* Removed, handled by gap in App.vue */
  min-width: 250px;
  max-width: 300px; /* Added max-width */
  background-color: #f9f9f9;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  border-radius: 4px;
}
.info-panel.placeholder {
  text-align: center;
  color: #777;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 150px; /* Give placeholder some height */
}
.info-panel div {
  margin-bottom: 12px;
}
label {
  display: block;
  margin-bottom: 4px;
  font-weight: bold;
  font-size: 0.9em;
  color: #333;
}
input[type="text"],
textarea,
select {
  width: 100%; /* Use 100% and box-sizing */
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 0.95em;
}
textarea {
  min-height: 70px;
  resize: vertical;
}
h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}
</style>
