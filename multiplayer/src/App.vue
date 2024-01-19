<script setup lang="ts">
import {computed, ref} from "vue"
import {type Ref} from "vue"
import MarioAnimation from './components/MarioAnimation.vue'

const serverUrl = 'wss://websocket-multiplayer-demo.onrender.com'
const socket = new WebSocket(serverUrl)
const isPlayer1 = computed(() => player.value === 1)
const isPlayer2 = computed(() => player.value === 2)
const clicks2 = ref(0)
const clicks1 = ref(0)
const player: Ref<number | undefined> = ref(undefined)
const state = ref('WAITING')
const winner: Ref<boolean | undefined> = ref(undefined)

const musicPlaying = ref(false)
const music = new Audio('/music.mp3')
const countDown = ref(3)
let countDownInterval = ref(false)
const runCountDown = () => {
  if(countDown.value > 0) {
    countDown.value--
  } else {
    clearInterval(countDownInterval.value)
    countDownInterval.value = false
  }
}

socket.addEventListener('message', function (event) {
  const data = JSON.parse(event.data)
  if(data.player) {player.value = data.player}
  if(data.clicks1) clicks1.value = data.clicks1
  if(data.clicks2) clicks2.value = data.clicks2
  if(data.state) state.value = data.state
  if(state.value === 'GAME_READY' || state.value === 'GAME_RUNNING') winner.value = undefined
  if(state.value === 'GAME_READY') {
    countDown.value = 3
    clicks1.value = 0
    clicks2.value = 0
    if(!countDownInterval.value) countDownInterval.value = setInterval(() => runCountDown(), 1000)
  }
  if(state.value === 'GAME_RUNNING' && !musicPlaying.value) {
    musicPlaying.value = true
    music.currentTime = 0
    music.play()
  }
  if(data.message === 'USER LEFT') location.reload()
  getWinner()
})

socket.addEventListener('open', function(){
  console.log('connected')
})

let clicks = ref(0)
const addClick = (id: string) => {
  if (countDown.value === 0) {
    clicks.value++;
    socket.send(id)
  }
}
const retryGame = () => {
  socket.send('RETRY')
}

function getWinner() {
  if(state.value === `PLAYER${player.value}_WON`) winner.value = true
  else if (state.value === 'PLAYER1_WON' || state.value === 'PLAYER2_WON') winner.value = false
  if(state.value !== 'GAME_RUNNING') musicPlaying.value = false
}
</script>

<template>
  <h1>Super Mario<br>Clicky Clicky</h1>
  <div class="scoreboard">
    <div>
      <div class="you" :class="{'hidden': isPlayer2}">&#128315;</div>
      <h2>P1</h2>
      <div class="scoredisplay">{{ clicks1 }}</div>
    </div>
    <div>
      <div class="you" :class="{'hidden': isPlayer1}">&#128315;</div>
      <h2>P2</h2>
      <div class="scoredisplay">{{ clicks2 }}</div>
    </div>
  </div>
  <div class="race-wrapper">
    <div class="countdown" :class="{'fade-out': countDown === 0}">{{ countDown }}</div>
    <div class="mario-wrapper">
      <MarioAnimation :is-player-2="false" :clicks="clicks1" />
      <MarioAnimation :is-player-2="true" :clicks="clicks2" />
    </div>
    <img class="finish-line" src="/finishline.png">
    <div class="finish"></div>
    <div class="message">
      <h2 v-if="winner === true">YOU WON!</h2>
      <h2 v-if="winner === false">YOU LOST!</h2>
      <button class="button" v-if="winner !== undefined" @click="retryGame">TRY AGAIN</button>
    </div>
  </div>
  <div class="player" v-if="state === 'GAME_READY' || state === 'GAME_RUNNING'">
    <button class="button" @click="addClick('click1')" v-if="player === 1">CLICK TO RUN</button>
    <button class="button" @click="addClick('click2')" v-if="player === 2">CLICK TO RUN</button>
  </div>
  <div class="loading" v-else>
    WAITING FOR PLAYER<span>.</span><span>.</span><span>.</span>
  </div>

</template>

<style scoped>
.scoreboard {
  display: flex;
  justify-content: center;
  gap: 16px;
  background: black;
  padding: 16px;
  color: white;
  width: 258px;
}

h1 {
  font-family: 'New Super Mario Font U', sans-serif;
  margin-bottom: 16px;
  color: white;
}

h2 {
  text-align: center;
  color: white;
  font-weight: bolder;
}
button, .scoredisplay, .you {
  font-weight: bolder;
  text-align: center;
}

.you {
  font-size: 16px;
  color: red;
}

.scoredisplay {
  font-size: 24px;
}

img {
  pointer-events: none;
  user-select: none;
  user-drag: none;
}

button {
  background: black;
  color: white;
  border: transparent;
  padding: 16px;
  font-weight: bolder;
  font-family: 'New Super Mario Font U';
}

.mario-wrapper {
  width: 184px;
  background: #378805;
}

.finish-line {
  height: 100%;
}
.finish {
  background: #378805;
  height: 100%;
  width: 50px;
}
.race-wrapper {
  display: flex;
  height: 150px;
  justify-content: flex-start;
  justify-self: center;
}

.button{
  position: relative;
  background-color: black;
  border-radius: 4em;
  font-size: 16px;
  color: white;
  padding: 0.8em 1.8em;
  cursor:pointer;
  user-select:none;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
  transition-duration: 0.4s;
  -webkit-transition-duration: 0.4s; /* Safari */
}

.button:hover {
  transition-duration: 0.1s;
  background-color: #3A3A3A;
}

.button:after {
  content: "";
  display: block;
  position: absolute;
  border-radius: 4em;
  left: 0;
  top:0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: all 0.5s;
  box-shadow: 0 0 10px 40px yellow;
}

.button:active:after {
  box-shadow: 0 0 0 0 yellow;
  position: absolute;
  border-radius: 4em;
  left: 0;
  top:0;
  opacity: 1;
  transition: 0s;
}

.button:active {
  top: 1px;
}

.message {
  display: flex;
  flex-direction: column;
  gap: 8px;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}

.player {
  margin-top: 16px;
}

.countdown {
  position: absolute;
  top: 10%;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1;
  font-size: 64px;
  color: red;
  text-shadow: -2px -2px 0 yellow, 2px -2px 0 yellow, -2px 2px 0 yellow, 2px 2px 0 yellow;
}

.hidden {
  opacity: 0;
}

.fade-out {
  visibility: hidden;
  opacity: 0;
  transition: visibility 0s 2s, opacity 2s linear;
}

@keyframes blink {
  0% {
    opacity: .2;
  }
  20% {
    opacity: 1;
  }
  100% {
    opacity: .2;
  }
}

.loading span {
  animation-name: blink;
  animation-duration: 1.4s;
  animation-iteration-count: infinite;
  animation-fill-mode: both;
}

.loading span:nth-child(2) {
  animation-delay: .2s;
}

.loading span:nth-child(3) {
  animation-delay: .4s;
}
</style>
