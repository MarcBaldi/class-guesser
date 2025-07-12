<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { Canvas, FabricImage, Text } from 'fabric'

const canvasRef = ref<HTMLCanvasElement | null>(null)
let canvas: Canvas

const classes = [
  'Guardian',
  'Warrior',
  'Revenant',
  'Engineer',
  'Thief',
  'Ranger',
  'Mesmer',
  'Necromancer',
  'Elementalist',
]

onMounted(async () => {
  if (!canvasRef.value) return

  // Canvas initialisieren
  canvas = new Canvas(canvasRef.value, {
    width: 800,
    height: 675
  })
  // Bild laden via Promise API
  for (let i = 0; i <= 9; i++) {

    const img = await FabricImage.fromURL(`/classes/${i}.png`)
    img.scaleToWidth(100)
    img.scaleToHeight(100)

    img.left = 25
    img.top = 25 + i * 75
    img.selectable = true
    img.hasControls = true

    canvas.add(img)
    canvas.setActiveObject(img)

    const columns = 3;
    const spacing = 150; // Abstand zwischen den Elementen

    const left = (i % columns) * spacing + 200; // Spalte (0,1,2) * Abstand + Offset
    const top = Math.floor(i / columns) * spacing + 10; // Zeile * Abstand + Offset


    const text = new Text(classes[i], {
      left: left, // Mitte über dem Bild
      top: top,
      fontSize: 18,
      fontWeight: 'bold',
      originX: 'center',
      selectable: false,
    })
    canvas.add(text)
  }
})

onBeforeUnmount(() => {
  // Ressourcen aufräumen
  canvas?.dispose()
})

</script>

<template>
  <h2>Class guesser</h2>
  <div>
    <canvas ref="canvasRef" width="800" height="600" style="border:1px solid #ccc;"></canvas>
  </div>
</template>

<style scoped>
canvas {
  border: 1px solid #ccc;
}
</style>