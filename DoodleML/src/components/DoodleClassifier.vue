<template>
  <div style="display: flex; flex-direction: column; align-items: center;">
    <div id="canvasContainer"></div>
    <h2>
      {{ label }}
    </h2>
    <button @click="clearCanvas">Clear canvas</button>
  </div>
</template>

<script setup>
import { imageClassifier } from 'ml5';
import p5 from 'p5';
import { ref, onMounted } from 'vue';

let classifier;
let sketchInstance;
let label = ref('');

let doodleClassifier;

function setupP5Instance(sketch) {
  sketch.setup = () => {
    const canvas = sketch.createCanvas(400, 400);
    canvas.parent('canvasContainer');
    sketch.background(255);
    doodleClassifier = imageClassifier('DoodleNet', modelReady);
  };

  sketch.draw = () => {
    if (sketch.mouseIsPressed) {
      sketch.strokeWeight(16);
      sketch.line(sketch.mouseX, sketch.mouseY, sketch.pmouseX, sketch.pmouseY);
      
      // Classify the current drawing
      doodleClassifier.classify(sketchInstance.canvas, gotResults);
    }
  };
}

function gotResults(error, results) {
  if (error) {
    console.error(error);
    return;
  }

  console.log(results);

  let content = `${results[0].label} 
  ${sketchInstance.nf(100 * results[0].confidence, 2, 1)}%
  ${results[1].label} ${sketchInstance.nf(
    100 * results[1].confidence,
    2,
    1
  )}%`;

  label.value = content;
}

async function modelReady() {
  console.log('Model ready');
}

function clearCanvas() {
  sketchInstance.background(255);
}

async function initClassifier() {
  classifier = await imageClassifier('DoodleNet', modelReady);
}

onMounted(async () => {
  new p5((sketch) => {
    sketchInstance = sketch;
    setupP5Instance(sketchInstance);
  });

  await initClassifier();
});
</script>
//CONVOLUTIONAL NEURAL NETWORK