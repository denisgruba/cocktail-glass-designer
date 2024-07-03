<template>
    <rect x="0" :y="`${fill.percentageStart}%`" width="100%" :height="`${fill.percentageHeight}%`" v-for="(fill, index) in fills" :style="{ fill: fill.color}"
        :key="index" />
</template>

<script setup lang="ts">
import { defineProps } from 'vue'

interface Fills {
    color: string
    percentageStart: number
    percentageHeight: number
}[]


const { fills } = defineProps({
    fills: {
        type: Array<Fills>,
        default: [],
    }
})

</script>
