<template>
    <Glass :liquids="liquids" :glass="selectedGlass"></Glass>
    <div>
        <div v-for="(_, index) in liquids">
            Fill percentage: <input type="number" v-model="liquids[index].height" />
            Color: <input type="color" v-model="liquids[index].color" />
        </div>

        <button @click="add">Add</button>
    </div>

<div>
    Glass
    <select v-model="selectedGlass">
        <option :value="glass" v-for="glass in glasses">{{ glass }}</option>
    </select>
</div>
</template>

<script setup lang="ts">
import Glass from './Glass.vue'
import { ref } from 'vue'

interface Liquid {
  color: string
  height: number
}

const liquids = ref<Liquid[]>([])

const add = () => {
    liquids.value.push({
        height: 10,
        color: "#100200",
    })
}

const glasses = [
    'Lowball',
    'Martini'
]

const selectedGlass = ref('Lowball')

</script>
