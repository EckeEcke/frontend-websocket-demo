<template>
  <div ref="marioBox" class="mario-box">
    <div class="position-relative">
      <img ref="marioSprite" class="mario-sprite"  :class="isPlayer2 ? 'player2' : ''" src="/mariosprites-walking.png" draggable="false">
    </div>
  </div>
</template>

<script setup lang="ts">
const props = defineProps({
  isPlayer2: Boolean,
  clicks: Number,
})
import { ref, watch } from 'vue'
const marioBox = ref()
const marioSprite = ref()
let marioLeft = 0

function movingBox(){
  marioBox.value.style.transform = `translateX(${marioLeft}px)`
  if(marioLeft < 200){
    marioLeft += 2
  }

  if(marioLeft >=200 || props.clicks === 0){
    marioLeft = 0
  }
}

let spritesheetPosition = 50
let sheetMovement = 50

const isRunning = ref(false)
function animateMario(){
  spritesheetPosition += sheetMovement;
  marioSprite.value.style.transform = `translateX(${-spritesheetPosition}px)`

  if(spritesheetPosition === 100){
    sheetMovement = -50
  }

  if(spritesheetPosition === 0){
    sheetMovement = 50
  }
}

let animationInterval: any
watch(() => props.clicks, (newValue, oldValue) => {
  movingBox()
  if(!isRunning.value && newValue && newValue > 0) {
    isRunning.value = true
    animationInterval = setInterval(animateMario, 102);
  }
  if(newValue === 0){
    clearInterval(animationInterval)
    isRunning.value = false
    animateMario()
    movingBox()
  }
})


</script>

<style scoped>
.mario-box {
  height: 60px;
  width: 50px;
  overflow: hidden;
  z-index: 5;
}

.position-relative {
  position: relative;
}

.mario-sprite {
  position: absolute;
  right: -100px;
  height: 60px;
  width: 150px;
  cursor: pointer;
}

.player2 {
  filter: sepia(90%);
}
</style>
