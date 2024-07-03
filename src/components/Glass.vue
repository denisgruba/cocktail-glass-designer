<template>
    <div class="glass">
        <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 500 500"
            v-if="glassConfig" ref="svgObjectRef">
            <defs>
                <clipPath id="liquid-clip-path">
                    <polygon :points="glassConfig.clipMask" ref="clipPathRef" />
                </clipPath>
            </defs>

            <defs>
                <clipPath id="glass-clip-path">
                    <path :d="glassConfig.glassShape" />
                </clipPath>
            </defs>

            <defs>
                <linearGradient gradientTransform="rotate(148, 0.5, 0.5)" x1="50%" y1="0%" x2="50%" y2="100%"
                    id="ffflux-gradient">
                    <stop stop-color="hsl(218, 46%, 67%)" stop-opacity="1" offset="0%"></stop>
                    <stop stop-color="hsl(227, 27%, 70%)" stop-opacity="1" offset="100%"></stop>
                </linearGradient>
                <filter id="ffflux-filter" x="-20%" y="-20%" width="140%" height="140%" filterUnits="objectBoundingBox"
                    primitiveUnits="userSpaceOnUse" color-interpolation-filters="sRGB">
                    <feTurbulence type="fractalNoise" baseFrequency="0.007 0.003" numOctaves="1" seed="2"
                        stitchTiles="stitch" x="0%" y="0%" width="100%" height="100%" result="turbulence">
                    </feTurbulence>
                    <feGaussianBlur stdDeviation="7 0" x="0%" y="0%" width="100%" height="100%" in="turbulence"
                        edgeMode="duplicate" result="blur"></feGaussianBlur>
                    <feBlend mode="screen" x="0%" y="0%" width="100%" height="100%" in="SourceGraphic" in2="blur"
                        result="blend"></feBlend>
                    <feColorMatrix type="saturate" values=".5" x="0%" y="0%" width="100%" height="100%" in="blend"
                        result="colormatrix"></feColorMatrix>

                </filter>
            </defs>
            <g id="Liquid" class="glass__clipped-group">
                <Liquid :fills="proportionalLiquids" />
            </g>
            <rect width="700" height="700" fill="url(#ffflux-gradient)" filter="url(#ffflux-filter)"
                class="glass__glass-body"></rect>
        </svg>
    </div>
</template>

<script setup lang="ts">
import Liquid from './Liquid.vue'
import { computed, ref, watch, nextTick, defineProps } from 'vue';


interface GlassConfig {
    clipMask: string
    glassShape: string
}

interface Liquid {
    color: string
    height: number
}

const props = defineProps({
    liquids: {
        type: Array<Liquid[]>,
        default: []
    },
    glass: {
        type: String,
        default: ''
    }
})

const glassName = computed(() => props.glass)

const clipPathRef = ref<null | Element>(null)
const svgObjectRef = ref<null | Element>(null)

const glassConfig = ref<GlassConfig | null>(null)

const clippedAreaDimensions = ref({
    top: 0,
    bottom: 0,
    height: 0
})

const svgAreaDimensions = ref({
    top: 0,
    bottom: 0,
    height: 0
})

const clippedAreaHeightRatio = computed(() => clippedAreaDimensions.value.height / svgAreaDimensions.value.height)

const topDelta = computed(() => clippedAreaDimensions.value.top - svgAreaDimensions.value.top)

const svgTopMarginRatio = computed(() => topDelta.value / svgAreaDimensions.value.height)

const bottomMarginPercent = computed(() => svgTopMarginRatio.value + clippedAreaHeightRatio.value)

const proportionalLiquids = computed(() => {

    let heightSoFar = 0

    return props.liquids?.reduce((total: any, current: any) => {
        const currentLiquidHeight = current.height * clippedAreaHeightRatio.value

        heightSoFar += currentLiquidHeight

        const percentageStart = (bottomMarginPercent.value * 100) - heightSoFar

        const processedCurrent = {
            ...current,
            percentageStart: percentageStart,
            percentageHeight: currentLiquidHeight
        }

        return [...total, processedCurrent]
    }, [])



})

watch(glassName, async () => {
    const { default: config } =
        await import(`../glasses/${glassName.value}.js`)


    glassConfig.value = config

    await nextTick()

    const { top: t1, bottom: b1, height: h1 } = clipPathRef.value?.getBoundingClientRect() as DOMRect
    clippedAreaDimensions.value = { top: t1, bottom: b1, height: h1 }

    const { top: t2, bottom: b2, height: h2 } = svgObjectRef.value?.getBoundingClientRect() as DOMRect
    svgAreaDimensions.value = { top: t2, bottom: b2, height: h2 }

}, {
    immediate: true
})


</script>

<style>
.glass {
    height: 500px;
    width: 500px;
}

.glass__clipped-group {
    clip-path: url(#liquid-clip-path);
}

.glass__glass-body {
    fill: #696d7dd9;
    clip-path: url(#glass-clip-path);
}
</style>
