<script setup lang="ts">
  import { ref } from 'vue'
  import { useLoop } from '@tresjs/core'
  import type { Mesh } from 'three'

  const dieRef = ref<Mesh | null>(null)

  const { onBeforeRender } = useLoop()

  onBeforeRender(({ delta }) => {
    if (dieRef.value) {
      dieRef.value.rotation.x += delta * 0.5
      dieRef.value.rotation.y += delta * 0.3
    }
  })
</script>

<template>
  <TresPerspectiveCamera
    :position="[7, 7, 7]"
    :look-at="[0, 0, 0]"
  />

  <TresMesh
    ref="dieRef"
    :position="[0, 2, 0]"
  >
    <TresIcosahedronGeometry />
    <!-- <TresMeshBasicMaterial color="#ff6b35" /> -->
    <TresMeshNormalMaterial />
  </TresMesh>

  <TresAxesHelper />
  <TresGridHelper />
</template>
