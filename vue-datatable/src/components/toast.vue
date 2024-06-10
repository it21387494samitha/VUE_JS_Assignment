<template>
    <div v-if="visible" class="toast" :class="type">{{ message }}</div>
  </template>
  
  <script>
  export default {
    props: ['message', 'type'],
    data() {
      return {
        visible: true
      };
    },
    mounted() {
      setTimeout(() => {
        this.visible = false;
        this.$emit('hidden');
      }, 3000); // Toast message will disappear after 3 seconds
    }
  };
  </script>
  
  <style scoped>
  .toast {
  position: fixed;
  top: 20px; 
  right: 20px; 
  padding: 10px 20px;
  border-radius: 5px;
  color: white;
  background-color: var(--toast-background-color, #333);
  opacity: 0.9;
}
  
  .toast.success {
    background-color: var(--toast-success-background-color, green);
  }
  
  .toast.error {
    background-color: var(--toast-error-background-color, red);
  }
  </style>
  