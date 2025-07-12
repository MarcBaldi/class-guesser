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
    width: 750,
    height: 625
  })

  const leftCoords = [
    25, 25, 25, 175, 325, 475, 625, 625, 625
  ]

  const topCoords = [
    50, 200, 350, 500, 500, 500, 350, 200, 50
  ]

// Texte laden
  const columns = 3;
  const spacing = 150; // Abstand zwischen den Elementen

  for (let i = 0; i <= 8; i++) {
    const left = (i % columns) * spacing + 225; // Spalte (0,1,2) * Abstand + Offset
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

  // Bild laden via Promise API
  for (let i = 0; i <= 8; i++) {

    const img = await FabricImage.fromURL(`/classes/${i}.png`)
    img.scaleToWidth(100)
    img.scaleToHeight(100)

    const leftImg = leftCoords[i]
    const topImg = topCoords[i]
    img.left = leftImg
    img.top = topImg
    img.selectable = true
    img.hasControls = true

    canvas.add(img)
    canvas.setActiveObject(img)
  }

  // Snap-Funktion
  const gridSize = 150;
  const leftOffset = 25;
  const topOffset = 50;
  function snapToGrid(obj) {

    obj.set({
      left: Math.min(Math.max(Math.round((obj.left - leftOffset) / gridSize) * gridSize + leftOffset, leftOffset), 625),
      top:  Math.min(Math.max(Math.round((obj.top - topOffset) / gridSize) * gridSize + topOffset, topOffset), 500)
    });
    obj.setCoords(); // ganz entscheidend!
  }

  // Wähle, wann gesnappt wird:
  canvas.on('object:moving', e => {
    snapToGrid(e.target);
  });
})

onBeforeUnmount(() => {
  // Ressourcen aufräumen
  canvas?.dispose()
})

</script>

<template>
  <h2 class="text-center">Class guesser</h2>
  <p class="text-center">
    <a href="https://bsky.app/profile/guildwars2.com">https://bsky.app/profile/guildwars2.com</a>
  </p>
  <div class="d-flex canvas-container">
    <canvas ref="canvasRef" width="800" height="600" style="border:1px solid #ccc;"></canvas>
  </div>
  <p><small>NCSOFT, the interlocking NC logo, ArenaNet, Guild Wars, Guild Wars Factions, Guild Wars Nightfall, Guild Wars: Eye of the North, Guild Wars 2, and all associated logos and designs are trademarks or registered trademarks of NCSOFT Corporation. All other trademarks are the property of their respective owners.
  </small></p>
</template>

<style scoped>
canvas {
  border: 1px solid #ccc;
}

.canvas-container {
  justify-content: center;
}
</style>
