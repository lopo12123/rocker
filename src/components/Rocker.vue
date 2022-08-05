<script lang="ts" setup>
import { computed, onBeforeUnmount, onMounted, ref } from "vue";

export interface RockerConfig {
    size?: number
    cubeClass?: string
    cubeSize?: number
    cubeRotate?: boolean
}

export type JobCanceller = () => void
export type JobRegistrar = (job: (normalizedOffset: { x: number, y: number }) => void, ms: number) => JobCanceller

const props = withDefaults(defineProps<RockerConfig>(), {
    size: 100,
    cubeClass: '',
    cubeSize: 20,
    cubeRotate: true,
})

const emits = defineEmits<{
    (ev: 'rocker-ready', loopRegistrar: JobRegistrar): void
}>()

const _normalizedOffset = ref({ x: 0, y: 0 })
const jobRegistrar: JobRegistrar = (job, ms) => {
    let cancelId = -1
    let last_t

    ;(function loop(time: number) {
        cancelId = requestAnimationFrame(loop)

        if(last_t === undefined) last_t = time
        else if(time - last_t >= ms) {
            job(_normalizedOffset.value)
            last_t = time
        }
    })(performance.now())

    return () => {
        cancelAnimationFrame(cancelId)
    }
}

const minmax = (init: number, min: number, max: number) => {
    return Math.max(min, Math.min(init, max))
}

const _size = computed(() => {
    return minmax(props.size, 100, 500)
})
const _cubeSize = computed(() => {
    return minmax(props.cubeSize, 20, 100)
})
const _cubeOffset = ref({ top: '50%', left: '50%' })

const rockerBox = ref<HTMLDivElement | null>(null)
const rockerCube = ref<HTMLDivElement | null>(null)

const getLimitedOffset = (x: number, x1: number, y: number, y1: number, limit: number) => {
    const dy = y1 - y
    const dx = x1 - x
    const dis = Math.sqrt(dx * dx + dy * dy)

    if(dis <= limit) return { left: x1, top: y1 }
    else {
        return {
            left: parseInt((x + limit * dx / dis).toFixed(0)),
            top: parseInt((y + limit * dy / dis).toFixed(0))
        }
    }
}

const mousedownCallback = (downEvent: MouseEvent) => {
    const boxSize = _size.value
    const xInBox = downEvent.offsetX
    const yInBox = downEvent.offsetY
    const xInScreen = downEvent.clientX
    const yInScreen = downEvent.clientY
    const centerInScreen = {
        top: yInScreen - yInBox + boxSize / 2,
        left: xInScreen - xInBox + boxSize / 2
    }

    if(downEvent.button === 0) {
        _cubeOffset.value = {
            top: yInBox + 'px',
            left: xInBox + 'px',
        }

        const mousemoveCallback = (moveEvent: MouseEvent) => {
            const offsetInScreen = getLimitedOffset(centerInScreen.left, moveEvent.clientX, centerInScreen.top, moveEvent.clientY, boxSize / 2)
            const offsetInBox = {
                top: offsetInScreen.top - (centerInScreen.top - boxSize / 2),
                left: offsetInScreen.left - (centerInScreen.left - boxSize / 2),
            }
            _cubeOffset.value = {
                top: offsetInBox.top + 'px',
                left: offsetInBox.left + 'px'
            }

            _normalizedOffset.value = {
                x: parseFloat((offsetInBox.left * 2 / boxSize - 1).toFixed(2)),
                y: parseFloat((offsetInBox.top * 2 / boxSize - 1).toFixed(2)),
            }
        }
        const mouseupCallback = () => {
            document.removeEventListener('mousemove', mousemoveCallback)
            document.removeEventListener('mouseup', mouseupCallback)
            _cubeOffset.value = {
                top: '50%',
                left: '50%'
            }
            _normalizedOffset.value = { x: 0, y: 0 }
        }
        document.addEventListener('mousemove', mousemoveCallback)
        document.addEventListener('mouseup', mouseupCallback)
    }
}

const setupRocker = () => {
    const box = rockerBox.value
    const cube = rockerCube.value

    if(!box || !cube) console.warn('box or cube is undefined!')
    else {
        box.addEventListener('mousedown', mousedownCallback)
        emits('rocker-ready', jobRegistrar)
        console.log('rocker has been initialized.')
    }
}
const disposeRocker = () => {
    rockerBox.value?.removeEventListener('mousedown', mousedownCallback)
    console.log('rocker has been disposed.')
}

onMounted(() => {
    setupRocker()
})
onBeforeUnmount(() => {
    disposeRocker()
})
</script>

<template>
    <div class="rocker-boundary" ref="rockerBox"
         :style="`width: ${_size}px; height: ${_size}px;`">
        <div :class="`rocker-cube ${props.cubeRotate ? 'rocker-cube__rotate' : ''} ${props.cubeClass}`"
             :style="`
             width: ${_cubeSize}px;
             height: ${_cubeSize}px;
             top: ${_cubeOffset.top};
             left: ${_cubeOffset.left};
             margin: -${_cubeSize / 2}px;
             `"
             ref="rockerCube">
        </div>
    </div>
</template>

<style lang="scss" scoped>
@keyframes rocker-cube__rotate {
    100% {
        transform: rotate(1turn);
    }
}

.rocker-cube__rotate {
    &::before {
        animation: 4s rocker-cube__rotate linear infinite;
    }
}

.rocker-boundary {
    position: relative;
    border: dashed 1px #7778;
    border-radius: 50%;
    flex-shrink: 0;
    box-sizing: border-box;

    > * {
        user-select: none;
        box-sizing: border-box;
    }

    .rocker-cube {
        position: absolute;
        border-radius: 50%;
        cursor: move;
        pointer-events: none;

        &::before {
            content: "";
            position: absolute;
            z-index: -2;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-repeat: no-repeat;
            background-size: 50% 50%;
            background-position: 0 0, 100% 0, 100% 100%, 0 100%;
            background-image: linear-gradient(-45deg, #399953cc, #3999531a),
            linear-gradient(45deg, #fbb300cc, #fbb3001a),
            linear-gradient(135deg, #6564f8cc, #6564f81a),
            linear-gradient(225deg, #d53e33cc, #d53e331a);
        }

        &::after {
            content: "";
            position: absolute;
            z-index: -1;
            width: calc(100% - 12px);
            height: calc(100% - 12px);
            left: 6px;
            top: 6px;
            border-radius: 50%;
            background: white;
        }
    }
}
</style>