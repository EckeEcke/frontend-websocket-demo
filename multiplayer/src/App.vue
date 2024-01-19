<script setup lang="ts">
import {ref} from "vue"
import {type Ref} from "vue"
import MarioAnimation from './components/MarioAnimation.vue'

const serverUrl = 'wss://websocket-multiplayer-demo.onrender.com'
const socket = new WebSocket(serverUrl)
const clicks2 = ref(0)
const clicks1 = ref(0)
const player = ref(undefined)
const state = ref('WAITING')
const winner: Ref<boolean | undefined> = ref(undefined)

const musicPlaying = ref(false)
const music = new Audio('/music.mp3')

socket.addEventListener('message', function (event) {
  const data = JSON.parse(event.data)
  if(data.player) {player.value = data.player}
  if(data.clicks1) clicks1.value = data.clicks1
  if(data.clicks2) clicks2.value = data.clicks2
  if(data.state) state.value = data.state
  if(state.value === 'GAME_READY' || state.value === 'GAME_RUNNING') winner.value = undefined
  if(state.value === 'GAME_READY') {
    clicks1.value = 0
    clicks2.value = 0
  }
  if(state.value === 'GAME_RUNNING' && !musicPlaying.value) {
    musicPlaying.value = true
    music.currentTime = 0
    music.play()

  }
  getWinner()
})

socket.addEventListener('open', function(){
  console.log('connected')
})

let clicks = ref(0)
const addClick = (id: string) => { clicks.value++; socket.send(id) }
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
      <h2>P1</h2>
      <div class="scoredisplay">{{ clicks1 }}</div>
    </div>
    <div>
      <h2>P2</h2>
      <div class="scoredisplay">{{ clicks2 }}</div>
    </div>
  </div>
  <div class="race-wrapper">
    <div class="mario-wrapper">
      <MarioAnimation :is-player-2="false" :clicks="clicks1" />
      <MarioAnimation :is-player-2="true" :clicks="clicks2" />
    </div>
    <img class="finish-line" src="/finishline.png#">
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
button, .scoredisplay {
  font-weight: bolder;
  text-align: center;
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
