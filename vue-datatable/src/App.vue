<template>
  <div :class="['app-container', { dark: isDarkMode }]">
    <button @click="toggleDarkMode" class="mode-toggle-button">
      {{ isDarkMode ? 'Switch to Light Mode' : 'Switch to Dark Mode' }}
    </button>
    <DataTable />
  </div>
</template>

<script>
import { ref } from 'vue';
import DataTable from './components/DataTable.vue';

export default {
  components: {
    DataTable,
  },
  setup() {
    const isDarkMode = ref(false);

    const toggleDarkMode = () => {
      isDarkMode.value = !isDarkMode.value;
      // Add/remove dark mode class to the root element and body
      if (isDarkMode.value) {
        document.documentElement.classList.add('dark');
        document.body.style.backgroundColor = '#2e2e2e';
      } else {
        document.documentElement.classList.remove('dark');
        document.body.style.backgroundColor = '#ffffff';
      }
    };

    return {
      isDarkMode,
      toggleDarkMode,
    };
  },
};
</script>

<style>
:root {
  --background-color: #ffffff;
  --text-color: #000000;
  --border-color: #ddd;
  --header-background-color: #f4f4f4;
  --action-button-background-color: #007bff;
  --action-button-hover-background-color: #0056b3;
  --delete-button-background-color: #dc3545;
  --save-button-background-color: #28a745;
  --save-button-hover-background-color:#218838;
}

.dark {
  --background-color: #2e2e2e;
  --text-color: #ffffff;
  --border-color: #444;
  --header-background-color: #3e3e3e;
  --action-button-background-color: #0056b3;
  --action-button-hover-background-color: #007bff;
  --delete-button-background-color: #b52b3d;
  --save-button-background-color: #28a745;
  --save-button-hover-background-color:#218838;
}

.app-container {
  background-color: var(--background-color);
  color: var(--text-color);
  min-height: 100vh;
  padding: 20px;
}

.mode-toggle-button {
  padding: 8px 16px;
  margin-bottom: 10px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  position: absolute;
}

.dark .mode-toggle-button {
  background-color: var(--action-button-background-color);
  color: var(--text-color);
}

.light .mode-toggle-button {
  background-color: var(--action-button-background-color);
  color: var(--text-color);
}
</style>
