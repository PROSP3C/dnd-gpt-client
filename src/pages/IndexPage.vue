<script setup lang="ts">
  import { TresCanvas } from '@tresjs/core'
  import TestComponent from '@/components/TestComponent.vue'
</script>

<template>
  <TresCanvas
    clear-color="#82DBC5"
    window-size
  >
    <TestComponent />
  </TresCanvas>
</template>
