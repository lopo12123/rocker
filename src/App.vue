<script lang="ts" setup>
import Rocker, { JobRegistrar } from "./components/Rocker.vue";
import { onBeforeUnmount, ref, shallowRef } from "vue";

const setupWorker = () => {
    const worker = new Worker('./raf.js')
    worker.onerror = (err) => {
        console.log(err)
    }
    console.log(worker)
}
</script>

<template>
    <div class="app">
        <button @click="setupWorker">setup worker</button>
    </div>
</template>

<style lang="scss">
.app {
    position: relative;
    width: 100%;
    height: 100%;
}
</style>