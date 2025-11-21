<script setup lang="ts">
  import { TresCanvas } from '@tresjs/core'
  import TestComponent from '@/components/TestComponent.vue'
  import { OrbitControls } from '@tresjs/cientos'
</script>

<template>
  <TresCanvas
    clear-color="#82DBC5"
    window-size
  >
    <TestComponent />
    <OrbitControls />
  </TresCanvas>
</template>
