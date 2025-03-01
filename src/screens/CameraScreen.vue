<template>
    <div class="h-screen w-full flex flex-col bg-gray-900 text-white">
        <header class="py-4 px-6 bg-gray-800 shadow-md">
            <h1 class="text-xl font-bold">Upload Profile Image</h1>
        </header>
        <main class="flex-1 overflow-hidden">
            <CameraComponent />
        </main>
    </div>
</template>

<script>
import { defineComponent } from 'vue';
import CameraComponent from '../components/CameraComponent.vue';

export default defineComponent({
    name: "CameraScreen",
    components: {
        CameraComponent
    }
})
</script>