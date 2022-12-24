<script lang="ts">
import Calculator from './components/Calculator.vue'

export default {
  components: {Calculator},
  setup() {
    const getRndInteger = (min: number, max: number): number => {
      return Math.floor(Math.random() * (max - min)) + min;
    }

    const allData = [
      {APY: 8, SOL_USD: 250},
      {APY: 3, SOL_USD: 300},
      {APY: 12, SOL_USD: 110},
      {APY: 25, SOL_USD: 376},
    ]

    const data = allData[getRndInteger(0, 4)]

    return {data}
  }
}
</script>

<template>
  <div class="wrapper">
    <Calculator :APY="data.APY" :SOL_USD="data.SOL_USD"/>
  </div>
</template>

<style scoped>
.wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
}
</style>
