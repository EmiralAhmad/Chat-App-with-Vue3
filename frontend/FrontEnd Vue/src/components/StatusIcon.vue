<template>
  <i class="icon" :class="{ connected: connected }"></i>
</template>

<script>
export default {
  name: "StatusIcon",
  props: {
    connected: Boolean,
  },
};
</script>

<style scoped>
.icon {
  height: 10px;
  width: 10px;
  border-radius: 50%;
  display: inline-block;
  background-color: #B7C4CF;
  margin-right: 8px;
}

.icon.connected {
  background-color: #3D8361;
}
</style>
