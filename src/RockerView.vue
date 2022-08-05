<script lang="ts" setup>
import Rocker, { JobRegistrar } from "./components/Rocker.vue";
import { onBeforeUnmount, ref, shallowRef } from "vue";

const playerOffset = ref({
    left: 250,
    top: 150
})

const canceller = shallowRef<Function | null>(null)
const doPlayerControl = (register: JobRegistrar) => {
    canceller.value = register(({ x, y }) => {
        playerOffset.value.top += y * 5
        playerOffset.value.left += x * 5
    }, 10)
}

onBeforeUnmount(() => {
    canceller.value?.()
})
</script>

<template>
    <div class="rocker-view">
        <div class="control">
            <Rocker @rocker-ready="doPlayerControl"/>
        </div>
        <div class="playground">
            <div class="ground">
                <div class="player"
                     :style="`top: ${playerOffset.top}px; left: ${playerOffset.left}px;`">😁
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
.rocker-view {
    position: relative;
    width: 100%;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;

    .control {
        width: fit-content;
        height: calc(100% - 40px);
        padding: 20px;
    }

    .playground {
        height: 100%;
        border-left: solid 1px #ccc;
        flex: 1;
        //display: flex;
        //align-items: center;
        //justify-content: center;

        .ground {
            width: 500px;
            height: 300px;
            border: solid 1px #ccc;

            .player {
                position: absolute;
                top: 50%;
                left: 50%;
                transition: all 0.2s;
            }
        }
    }
}
</style>