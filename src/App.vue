<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import { StatusBar, Style } from '@capacitor/status-bar'
import { Capacitor } from '@capacitor/core'

import CameraScreen from './screens/CameraScreen.vue'
import FileUploadScreen from './screens/FileUploadScreen.vue'

const showCamera = ref(false)
const showFileUpload = ref(true)

onMounted(async () => {
  if (Capacitor.isNativePlatform()) {
    try {
      // Make status bar transparent with dark content
      await StatusBar.setStyle({ style: Style.Dark });
      
      // This is crucial - make the status bar overlay the WebView
      if (Capacitor.getPlatform() === 'ios') {
        StatusBar.setOverlaysWebView({ overlay: true });
      } else {
        // For Android, set status bar color to match header
        await StatusBar.setBackgroundColor({ color: '#1f2937' }); // Tailwind gray-800
      }
    } catch (error) {
      console.error('Error configuring status bar:', error);
    }
  }
});
</script>

<template>
  <!-- Simplified root container -->
  <div class="app-root">
    <FileUploadScreen v-if="showFileUpload" />
    <CameraScreen v-if="showCamera" />
  </div>
</template>

<style>
/* Global styles */
:root {
  --safe-area-top: env(safe-area-inset-top, 0px);
  --safe-area-bottom: env(safe-area-inset-bottom, 0px);
  --safe-area-left: env(safe-area-inset-left, 0px);
  --safe-area-right: env(safe-area-inset-right, 0px);
}

body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
}

/* Fix for iOS notch */
body {
  position: fixed;
  width: 100%;
}
</style>

<style scoped>
.app-root {
  display: flex;
  flex-direction: column;
  height: 100vh;
  width: 100%;
}
</style>