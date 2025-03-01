<template>
  <div class="relative h-full w-full flex flex-col bg-neutral-950 overflow-hidden pb-16" style="height: 100vh;">
    <div v-if="!photo" class="h-full w-full flex flex-col relative">
      <video ref="videoElement" autoplay playsinline class="h-full w-full object-cover"></video>
      <div class="absolute bottom-8 left-0 right-0 flex justify-center items-center space-x-6">
        <button @click="takePicture" class="w-16 h-16 rounded-full bg-white bg-opacity-30 flex items-center justify-center">
          <div class="w-14 h-14 rounded-full bg-white"></div>
        </button>
        <button @click="switchCamera" class="w-12 h-12 rounded-full bg-neutral-950 bg-opacity-50 text-white flex items-center justify-center">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
          </svg>
        </button>
      </div>
    </div>
    <div v-else class="h-full w-full flex flex-col relative">
      <img :src="photo" alt="Captured photo" class="h-full w-full object-contain" />
      <div class="absolute bottom-8 left-0 right-0 flex justify-center items-center space-x-6">
        <button @click="retakePicture" class="px-6 py-2 rounded-lg text-white font-medium bg-red-500">Retake</button>
        <button @click="savePicture" class="px-6 py-2 rounded-lg text-white font-medium bg-green-500">Save</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { Camera, CameraResultType, CameraSource, CameraDirection } from '@capacitor/camera';

export default {
  name: 'CameraComponent',
  
  setup() {
    const videoElement = ref(null);
    const photo = ref(null);
    const mediaStream = ref(null);
    const currentCameraDirection = ref(CameraDirection.Rear);

    // For web browser implementation
    const startBrowserCamera = async () => {
      try {
        // Stop any existing stream
        if (mediaStream.value) {
          mediaStream.value.getTracks().forEach(track => track.stop());
        }

        // Get user media with specified camera
        const constraints = {
          video: {
            facingMode: currentCameraDirection.value === CameraDirection.Front ? 'user' : 'environment'
          }
        };
        
        mediaStream.value = await navigator.mediaDevices.getUserMedia(constraints);
        
        // Connect the stream to the video element
        if (videoElement.value) {
          videoElement.value.srcObject = mediaStream.value;
        }
      } catch (error) {
        console.error('Error accessing camera:', error);
        alert('Error accessing camera. Please check permissions.');
      }
    };

    // Handle taking picture in browser
    const takeBrowserPicture = () => {
      if (!videoElement.value) return;
      
      // Create a canvas element
      const canvas = document.createElement('canvas');
      canvas.width = videoElement.value.videoWidth;
      canvas.height = videoElement.value.videoHeight;
      
      // Draw the current video frame to the canvas
      const ctx = canvas.getContext('2d');
      ctx.drawImage(videoElement.value, 0, 0, canvas.width, canvas.height);
      
      // Convert canvas to data URL
      photo.value = canvas.toDataURL('image/jpeg');
      
      // Stop the camera stream
      if (mediaStream.value) {
        mediaStream.value.getTracks().forEach(track => track.stop());
        mediaStream.value = null;
      }
    };

    // Take picture using Capacitor Camera API (for mobile)
    const takeCapacitorPicture = async () => {
      try {
        const image = await Camera.getPhoto({
          quality: 90,
          allowEditing: false,
          resultType: CameraResultType.DataUrl,
          source: CameraSource.Camera,
          direction: currentCameraDirection.value
        });
        
        photo.value = image.dataUrl;
      } catch (error) {
        console.error('Error taking picture:', error);
        if (error.message !== 'User cancelled photo capture') {
          alert('Error taking picture. Please try again.');
        }
      }
    };

    // Detect if running in a Capacitor app
    const isCapacitorNative = () => {
      return typeof window !== 'undefined' && 
             window.Capacitor && 
             window.Capacitor.isNativePlatform();
    };

    // Public methods
    const takePicture = () => {
      if (isCapacitorNative()) {
        takeCapacitorPicture();
      } else {
        takeBrowserPicture();
      }
    };

    const retakePicture = () => {
      photo.value = null;
      if (!isCapacitorNative()) {
        startBrowserCamera();
      }
    };

    const savePicture = () => {
      // Here you would typically save the photo.value (data URL) to your application
      // This could involve sending it to a server, storing it locally, etc.
      alert('Photo saved!');
      photo.value = null;
      if (!isCapacitorNative()) {
        startBrowserCamera();
      }
    };

    const switchCamera = async () => {
      // Toggle camera direction
      currentCameraDirection.value = 
        currentCameraDirection.value === CameraDirection.Front 
          ? CameraDirection.Rear 
          : CameraDirection.Front;
      
      if (!isCapacitorNative()) {
        await startBrowserCamera();
      }
      // For Capacitor, the direction will be used next time takeCapacitorPicture is called
    };

    onMounted(() => {
      if (!isCapacitorNative()) {
        startBrowserCamera();
      }
    });

    onBeforeUnmount(() => {
      // Clean up camera resources
      if (mediaStream.value) {
        mediaStream.value.getTracks().forEach(track => track.stop());
        mediaStream.value = null;
      }
    });

    return {
      videoElement,
      photo,
      takePicture,
      retakePicture,
      savePicture,
      switchCamera
    };
  }
}
</script>

<style scoped>
/* No @apply directives needed - all styles are inline in the template */
</style>