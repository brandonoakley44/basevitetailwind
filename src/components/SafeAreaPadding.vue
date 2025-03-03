<script setup>
defineProps({
  top: {
    type: Boolean,
    default: true
  },
  bottom: {
    type: Boolean,
    default: true
  },
  left: {
    type: Boolean,
    default: true
  },
  right: {
    type: Boolean,
    default: true
  }
})
</script>

<template>
  <div class="safe-area-padding" :class="{
    'pad-top': top,
    'pad-bottom': bottom,
    'pad-left': left,
    'pad-right': right
  }">
    <slot></slot>
  </div>
</template>

<style scoped>
.safe-area-padding {
  display: contents;
}

.pad-top {
  padding-top: var(--safe-area-top, 0px);
}

.pad-bottom {
  padding-bottom: var(--safe-area-bottom, 0px);
}

.pad-left {
  padding-left: var(--safe-area-left, 0px);
}

.pad-right {
  padding-right: var(--safe-area-right, 0px);
}
</style>