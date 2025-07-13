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

const classesBg = [
  '#72C1D9',
  '#FFD166',
  '#D16E5A',
  '#D09C59',
  '#C08F95',
  '#8CDC82',
  '#B679D5',
  '#52A76F',
  '#F68A87',
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
      backgroundColor: classesBg[i]
    })
    canvas.add(text)

  }
  let hObj = [null, null, null, null, null, null, null, null, null]
  let imgPositions = new Array(20)
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
    img.hasControls = false

    canvas.add(img)
    canvas.setActiveObject(img)

    hObj[i] = img
    //coords[i] = { left: leftImg, top: topCoords }

  }

  // Snap-Funktion
  const gridSize = 150;
  const leftOffset = 25;
  const topOffset = 50;
  function snapToGrid(obj) {
    if (hObj == obj) {
      console.log("uhoh")
    }
    const snapLeft = Math.min(Math.max(Math.round((obj.left - leftOffset) / gridSize) * gridSize + leftOffset, leftOffset), 625)
    const snapTop = Math.min(Math.max(Math.round((obj.top - topOffset) / gridSize) * gridSize + topOffset, topOffset), 500)
    obj.set({
      left: snapLeft,
      top: snapTop
    });
    obj.setCoords(); // ganz entscheidend!

    const zielPos = ((snapTop - 50)  / 150 * 5 + (snapLeft - 25) / 150)
    imgPositions[zielPos] = obj

    //console.log(zielPos)

    /*hObj.forEach((h)=>{
      if (h.left == obj.left && h.top == obj.top) {
        console.log(h)
        console.log(obj)

        h.set({
          left: 0,
          top: 0
        });
      }
    })*/



  }

  // Wähle, wann gesnappt wird:
  canvas.on('object:modified', e => {
    snapToGrid(e.target);
  });
})

onBeforeUnmount(() => {
  // Ressourcen aufräumen
  canvas?.dispose()
})

const copyToClip = async function () {
  try {
    // Canvas in Blob konvertieren
    const blob = await new Promise(resolve =>
        canvas.toCanvasElement().toBlob(resolve, 'image/png')
    );

    // In Zwischenablage schreiben (ClipboardItem)
    await navigator.clipboard.write([
      new ClipboardItem({ 'image/png': blob })
    ]);

    alert('Bild wurde in die Zwischenablage kopiert!');
  } catch (err) {
    console.error('Fehler beim Kopieren in die Zwischenablage:', err);
    alert('Kopieren nicht möglich. Browser unterstützt evtl. keine Bild-Zwischenablage.');
  }
}

</script>

<template>
  <h2 class="text-center">Class guesser</h2>
  <p class="text-center">
    <span>Event: </span>
    <a href="https://bsky.app/profile/guildwars2.com">https://bsky.app/profile/guildwars2.com</a>
  </p>
  <div class="d-flex canvas-container justify-center">
    <canvas ref="canvasRef" width="800" height="600" style="border:1px solid #ccc;"></canvas>
  </div>
  <div class="d-flex justify-center">
    <v-btn title="Coming soon" disabled>Save</v-btn>
    <v-btn @click="copyToClip">Copy</v-btn>
  </div>
  <p><small>NCSOFT, the interlocking NC logo, ArenaNet, Guild Wars, Guild Wars Factions, Guild Wars Nightfall, Guild Wars: Eye of the North, Guild Wars 2, and all associated logos and designs are trademarks or registered trademarks of NCSOFT Corporation. All other trademarks are the property of their respective owners.
  </small></p>
</template>

<style scoped>
canvas {
  border: 1px solid #ccc;
}
</style>
