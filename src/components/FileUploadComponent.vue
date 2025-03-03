<template>
  <div class="file-upload-container p-4">
    <div class="mb-6">
      <h2 class="text-xl font-bold mb-2">File Upload</h2>
      <p class="text-gray-600 mb-4">Select files from your device to upload</p>
      
      <!-- Native file picker button -->
      <button 
        @click="pickFiles" 
        class="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded transition duration-200"
      >
        Select Files
      </button>
      
      <!-- Web fallback for direct file input -->
      <input 
        v-if="!isNativePlatform" 
        type="file" 
        ref="fileInput" 
        @change="handleFileSelection" 
        multiple
        class="hidden"
      />
    </div>
    
    <!-- Selected files display -->
    <div v-if="selectedFiles.length > 0" class="selected-files mb-6">
      <h3 class="text-lg font-semibold mb-2">Selected Files ({{ selectedFiles.length }})</h3>
      <div class="bg-gray-100 rounded p-3">
        <div 
          v-for="(file, index) in selectedFiles" 
          :key="index"
          class="flex justify-between items-center p-2 border-b border-gray-200 last:border-0"
        >
          <div class="flex items-center">
            <div class="file-icon mr-2">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                <polyline points="14 2 14 8 20 8"></polyline>
                <line x1="16" y1="13" x2="8" y2="13"></line>
                <line x1="16" y1="17" x2="8" y2="17"></line>
                <polyline points="10 9 9 9 8 9"></polyline>
              </svg>
            </div>
            <div class="file-info">
              <p class="file-name font-medium truncate max-w-xs">{{ file.name }}</p>
              <p class="file-size text-sm text-gray-500">{{ formatFileSize(file.size) }}</p>
            </div>
          </div>
          <button 
            @click="removeFile(index)" 
            class="text-red-500 hover:text-red-700"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </div>
      </div>
    </div>
    
    <!-- Upload actions -->
    <div v-if="selectedFiles.length > 0" class="upload-actions">
      <button 
        @click="uploadFiles" 
        class="bg-green-500 hover:bg-green-600 text-white font-medium py-2 px-4 rounded mr-2 transition duration-200"
        :disabled="isUploading"
      >
        <span v-if="isUploading">
          Uploading... {{ uploadProgress }}%
        </span>
        <span v-else>
          Upload Files
        </span>
      </button>
      <button 
        @click="clearFiles" 
        class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-medium py-2 px-4 rounded transition duration-200"
        :disabled="isUploading"
      >
        Clear All
      </button>
    </div>
    
    <!-- Upload progress -->
    <div v-if="isUploading" class="upload-progress mt-4">
      <div class="w-full bg-gray-200 rounded-full h-2.5">
        <div 
          class="bg-blue-600 h-2.5 rounded-full" 
          :style="{ width: `${uploadProgress}%` }"
        ></div>
      </div>
    </div>
    
    <!-- Upload results -->
    <div v-if="uploadResults.length > 0" class="upload-results mt-6">
      <h3 class="text-lg font-semibold mb-2">Upload Results</h3>
      <div class="bg-gray-100 rounded p-3">
        <div 
          v-for="(result, index) in uploadResults" 
          :key="index"
          class="p-2 border-b border-gray-200 last:border-0"
          :class="{'text-green-700': result.success, 'text-red-700': !result.success}"
        >
          <p><span class="font-medium">{{ result.fileName }}:</span> {{ result.message }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import { Capacitor } from '@capacitor/core';
import { FilePicker } from '@capawesome/capacitor-file-picker';
import { Filesystem, Directory } from '@capacitor/filesystem';

export default {
  name: 'FileUploadComponent',
  
  setup() {
    const fileInput = ref(null);
    const selectedFiles = ref([]);
    const isUploading = ref(false);
    const uploadProgress = ref(0);
    const uploadResults = ref([]);
    
    // Check if running on a native platform (iOS/Android)
    const isNativePlatform = computed(() => {
      return Capacitor.isNativePlatform();
    });
    
    // Native file picker for mobile
    const pickFiles = async () => {
      if (isNativePlatform.value) {
        try {
          const result = await FilePicker.pickFiles({
            multiple: true,
            readData: true
          });
          
          // Process the selected files
          if (result.files && result.files.length > 0) {
            result.files.forEach(file => {
              selectedFiles.value.push({
                name: file.name,
                path: file.path,
                data: file.data,
                mimeType: file.mimeType,
                size: file.size,
                isNative: true
              });
            });
          }
        } catch (error) {
          console.error('Error picking files:', error);
        }
      } else {
        // For web, trigger the hidden file input
        fileInput.value.click();
      }
    };
    
    // Handle file selection from web input
    const handleFileSelection = (event) => {
      const files = event.target.files;
      if (files && files.length > 0) {
        for (let i = 0; i < files.length; i++) {
          const file = files[i];
          selectedFiles.value.push({
            name: file.name,
            file: file,
            size: file.size,
            type: file.type,
            isNative: false
          });
        }
      }
    };
    
    // Remove a file from the selection
    const removeFile = (index) => {
      selectedFiles.value.splice(index, 1);
    };
    
    // Clear all selected files
    const clearFiles = () => {
      selectedFiles.value = [];
      uploadResults.value = [];
      uploadProgress.value = 0;
      
      // Reset web file input if it exists
      if (fileInput.value) {
        fileInput.value.value = null;
      }
    };
    
    // Format file size to human-readable format
    const formatFileSize = (bytes) => {
      if (bytes === 0) return '0 Bytes';
      
      const k = 1024;
      const sizes = ['Bytes', 'KB', 'MB', 'GB', 'TB'];
      const i = Math.floor(Math.log(bytes) / Math.log(k));
      
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    };
    
    // Simulated file upload
    const uploadFiles = async () => {
      if (selectedFiles.value.length === 0) return;
      
      isUploading.value = true;
      uploadProgress.value = 0;
      uploadResults.value = [];
      
      // Simulate backend upload process
      const totalFiles = selectedFiles.value.length;
      let completedFiles = 0;
      
      for (const file of selectedFiles.value) {
        // Simulate file processing and network delay
        await new Promise(resolve => setTimeout(resolve, 500 + Math.random() * 1000));
        
        try {
          // For native platforms, save the file to app storage to simulate upload
          if (isNativePlatform.value) {
            if (file.data) {
              // If we have file data, write it to the filesystem
              await Filesystem.writeFile({
                path: file.name,
                data: file.data,
                directory: Directory.Cache
              });
              
              uploadResults.value.push({
                fileName: file.name,
                success: true,
                message: 'File uploaded successfully!'
              });
            } else if (file.path) {
              // If we have a file path, copy it to our app storage
              const fileInfo = await Filesystem.stat({
                path: file.path
              });
              
              uploadResults.value.push({
                fileName: file.name,
                success: true,
                message: 'File uploaded successfully! Size: ' + formatFileSize(fileInfo.size)
              });
            }
          } else {
            // For web, we'd typically use FormData and fetch/axios to upload
            // Here we just simulate success
            uploadResults.value.push({
              fileName: file.name,
              success: true,
              message: 'File uploaded successfully!'
            });
          }
        } catch (error) {
          console.error('Error uploading file:', error);
          
          uploadResults.value.push({
            fileName: file.name,
            success: false,
            message: 'Upload failed: ' + (error.message || 'Unknown error')
          });
        }
        
        // Update progress
        completedFiles++;
        uploadProgress.value = Math.round((completedFiles / totalFiles) * 100);
      }
      
      // Show upload completion alert
      setTimeout(() => {
        isUploading.value = false;
        alert(`Upload complete! ${uploadResults.value.filter(r => r.success).length} of ${totalFiles} files uploaded successfully.`);
      }, 500);
    };
    
    return {
      fileInput,
      selectedFiles,
      isUploading,
      uploadProgress,
      uploadResults,
      isNativePlatform,
      pickFiles,
      handleFileSelection,
      removeFile,
      clearFiles,
      uploadFiles,
      formatFileSize
    };
  }
};
</script>
