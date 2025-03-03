<template>
  <div class="file-upload-test-page flex flex-col h-full">
    <!-- Header with negative margin to extend into safe area -->
    <header class="bg-gray-800 text-white py-4 px-6 shadow-md relative">
      <!-- This spacer accounts for the safe area -->
      <div class="w-full pt-safe" aria-hidden="true"></div>
      <h1 class="text-xl font-bold">File Upload Screen</h1>
    </header>
    <main class="p-4 flex-1">
      <FileUploadComponent />
    </main>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import FileUploadComponent from '../components/FileUploadComponent.vue';

export default defineComponent({
  name: 'FileUploadScreen',
  components: {
    FileUploadComponent
  }
});
</script>

<style>
/* Add this to your global CSS or keep it component-specific */
.pt-safe {
  padding-top: env(safe-area-inset-top);
}

.pb-safe {
  padding-bottom: env(safe-area-inset-bottom);
}
</style>