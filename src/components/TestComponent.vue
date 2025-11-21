<script setup lang="ts">
  import { onMounted, ref } from 'vue'
  import { useLoop } from '@tresjs/core'
  import type { Mesh } from 'three'
  import { Edges } from '@tresjs/cientos'
  import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
  import type { Font } from 'three/examples/jsm/loaders/FontLoader.js'
  import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js'
  import * as THREE from 'three'

  function getFaces(geometry: THREE.BufferGeometry) {
    const pos = geometry.attributes.position
    const faces: THREE.Vector3[][] = []

    if (!pos) return []

    if (geometry.index) {
      // Indexed geometry: use index array
      const index = geometry.index.array
      for (let i = 0; i < index.length; i += 3) {
        const a = index[i]
        const b = index[i + 1]
        const c = index[i + 2]

        if (a && b && c) {
          faces.push([
            new THREE.Vector3().fromBufferAttribute(pos, a),
            new THREE.Vector3().fromBufferAttribute(pos, b),
            new THREE.Vector3().fromBufferAttribute(pos, c),
          ])
        }
      }
    } else {
      // Non-indexed geometry: positions are sequential
      for (let i = 0; i < pos.count; i += 3) {
        faces.push([
          new THREE.Vector3().fromBufferAttribute(pos, i),
          new THREE.Vector3().fromBufferAttribute(pos, i + 1),
          new THREE.Vector3().fromBufferAttribute(pos, i + 2),
        ])
      }
    }

    return faces
  }

  const dieRef = ref<Mesh | null>(null)

  const { onBeforeRender } = useLoop()

  onMounted(() => {
    const loader = new FontLoader()
    const mesh = dieRef.value as Mesh
    const faces = getFaces(mesh.geometry)

    const d20Map: Record<number, string> = {
      0: '20',
      1: '2',
      2: '12',
      3: '10',
      4: '8',
      5: '18',
      6: '14',
      7: '16',
      8: '17',
      9: '15',
      10: '11',
      11: '9.',
      12: '19',
      13: '1',
      14: '13',
      15: '4',
      16: '6.',
      17: '3',
      18: '7',
      19: '5',
    }

    loader.load('/fonts/helvetiker_regular.typeface.json', (font: Font) => {
      faces.forEach((face, i) => {
        // Compute face center
        const center = new THREE.Vector3()
          .add(face[0] as THREE.Vector3)
          .add(face[1] as THREE.Vector3)
          .add(face[2] as THREE.Vector3)
          .divideScalar(3)

        // Compute face normal
        const edge1 = new THREE.Vector3().subVectors(
          face[1] as THREE.Vector3,
          face[0] as THREE.Vector3,
        )
        const edge2 = new THREE.Vector3().subVectors(
          face[2] as THREE.Vector3,
          face[0] as THREE.Vector3,
        )
        const normal = new THREE.Vector3()
          .crossVectors(edge1, edge2)
          .normalize()

        // Create text geometry
        const textGeometry = new TextGeometry(String(d20Map[i]), {
          font,
          size: 0.3,
          depth: 0.01,
          curveSegments: 12,
          bevelEnabled: false,
        })
        textGeometry.computeBoundingBox()
        textGeometry.center()

        const bbox = textGeometry.boundingBox

        if (bbox) {
          const textOffset = new THREE.Vector3(
            (bbox.max.x + bbox.min.x) / -2,
            (bbox.max.y + bbox.min.y) / -2,
            (bbox.max.z + bbox.min.z) / -2,
          )
          textGeometry.translate(textOffset.x, textOffset.y, textOffset.z)
        }

        const textMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff })
        const textMesh = new THREE.Mesh(textGeometry, textMaterial)

        // Position at face center + slight offset
        textMesh.position.copy(center).addScaledVector(normal, 0.01)

        // Orient text flat on the face
        const quat = new THREE.Quaternion()
        quat.setFromUnitVectors(new THREE.Vector3(0, 0, 1), normal)
        textMesh.quaternion.copy(quat)

        mesh.add(textMesh)
      })
    })
  })

  onBeforeRender(() => {
    // if (dieRef.value) {
    //   dieRef.value.rotation.x += delta * 0.5
    //   dieRef.value.rotation.y += delta * 0.3
    // }
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
    <TresMeshBasicMaterial
      color="#ff6b35"
      transparent
      :opacity="0.9"
    />
    <Edges color="#ffffff" />
  </TresMesh>

  <TresAxesHelper />
  <TresGridHelper />
</template>
