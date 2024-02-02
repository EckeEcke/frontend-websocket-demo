<script setup lang="ts">
import {computed, ref} from "vue"
import {type Ref} from "vue"
import MarioAnimation from './components/MarioAnimation.vue'

const serverUrl = 'wss://websocket-multiplayer-demo.onrender.com'
const socket = new WebSocket(serverUrl)
const connectedToSocket = ref(false)
const isPlayer1 = computed(() => player.value === 1)
const isPlayer2 = computed(() => player.value === 2)
const clicks2 = ref(0)
const clicks1 = ref(0)
const player: Ref<number | undefined> = ref(undefined)
const state = ref('WAITING')
const winner: Ref<boolean | undefined> = ref(undefined)

const musicPlaying = ref(false)
const music = new Audio('/music.mp3')
const winMusic = new Audio('/victory.wav')
const loseMusic = new Audio('/lose.wav')
const countDown = ref(3)
let countDownInterval: any = ref(false)
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
  connectedToSocket.value = true
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
  if(state.value === `PLAYER${player.value}_WON`) {
    winner.value = true
    winMusic.play()
  }
  else if (state.value === 'PLAYER1_WON' || state.value === 'PLAYER2_WON') {
    winner.value = false
    loseMusic.play()
  }
  if(state.value !== 'GAME_RUNNING') musicPlaying.value = false
}
</script>

<template>
  <div class="game-container">
    <h1>Super Mario<br>Clicky Clicky</h1>
    <div class="scoreboard">
      <div :class="{'current-player': isPlayer1}">
        <h2>PLAYER1:</h2>
        <div class="scoredisplay">{{ clicks1 }}</div>
      </div>
      <div :class="{'current-player': isPlayer2}">
        <h2>PLAYER2:</h2>
        <div class="scoredisplay">{{ clicks2 }}</div>
      </div>
    </div>
    <div class="race-wrapper">
      <div class="countdown highlight-font" :class="{'fade-out': countDown === 0, 'hidden': state === 'WAITING'}">{{ countDown }}</div>
      <div class="mario-wrapper">
        <MarioAnimation :is-player-2="false" :clicks="clicks1" />
        <div class="white-separator" />
        <MarioAnimation :is-player-2="true" :clicks="clicks2" />
      </div>
      <img class="finish-line" src="/finishline.png">
      <div class="finish">
        <div class="white-separator" />
      </div>
      <div class="message">
        <h2 v-if="winner === true" class="highlight-font">YOU WON</h2>
        <h2 v-if="winner === false" class="highlight-font">YOU LOST</h2>
        <button class="button" v-if="winner !== undefined" @click="retryGame">PLAY AGAIN</button>
      </div>
    </div>
    <div class="player" v-if="state === 'GAME_READY' || state === 'GAME_RUNNING'">
      <button class="button" @click="addClick('click1')" v-if="player === 1">CLICK TO RUN</button>
      <button class="button" @click="addClick('click2')" v-if="player === 2">CLICK TO RUN</button>
    </div>
    <div class="loading" v-else>
      {{ connectedToSocket ? 'WAITING FOR PLAYER' : 'WAITING FOR SERVER' }}<span>.</span><span>.</span><span>.</span>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.scoreboard {
  display: flex;
  justify-content: center;
  gap: 16px;
  background: black;
  padding: 16px;
  color: white;
  width: 258px;

  & div {
    display: flex;
    gap: 4px;
  }

  & .current-player {
    border-bottom: 4px solid white;
     & .scoredisplay,
        h2 {
          color: var(--orange);
     }
  }
}

h1 {
  font-family: 'New Super Mario Font U', sans-serif;
  text-align: center;
  margin-bottom: 16px;
  color: var(--red);
  text-shadow: -2px -2px 0 yellow, 2px -2px 0 yellow, -2px 2px 0 yellow, 2px 2px 0 yellow;
}

h2 {
  text-align: center;
  color: white;
  font-weight: bolder;
  font-size: 18px;
  line-height: 1;
}

.scoredisplay {
  font-size: 18px;
  font-weight: bolder;
  line-height: 1;
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
  border-top: 4px solid white;
  border-bottom: 4px solid white;
  background: var(--orange);
}

.white-separator {
  height: 24px;
  width: 100%;
  border-top: 4px solid white;
  border-bottom: 4px solid white;
  background: var(--green);
}

.finish-line {
  height: 100%;
  width: 24px;
}
.finish {
  display: flex;
  align-items: center;
  background: var(--orange);
  border-top: 4px solid white;
  border-bottom: 4px solid white;
  height: 100%;
  width: 50px;
}
.race-wrapper {
  display: flex;
  height: 168px;
  justify-content: flex-start;
  justify-self: center;
  border-bottom: 8px solid var(--green);
  border-top: 8px solid var(--green);
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
  gap: 16px;
  position: absolute;
  top: 50%;
  left: 50%;
  z-index: 10;
  transform: translate(-50%, -50%);
  text-align: center;
  width: 210px;
}

.player {
  margin-top: 16px;
  display: flex;
  justify-content: center;
}

.highlight-font {
  font-size: 32px;
  color: var(--red);
  text-shadow: -2px -2px 0 yellow, 2px -2px 0 yellow, -2px 2px 0 yellow, 2px 2px 0 yellow;
}

.countdown {
  position: absolute;
  top: 10%;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1;
  font-size: 64px;
}

.hidden {
  opacity: 0;
}

.fade-out {
  visibility: hidden;
  opacity: 0;
  transition: visibility 0s 1s, opacity 1s linear;
}

.loading {
  margin-top: 8px;
  text-align: center;
  color: white;
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

.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

@media (min-width: 800px) {
  .game-container {
    margin-top: 20vh;
    transform: scale(1.6);
    transform-origin: center;
  }
}

@media (min-width: 1400px) {
  .game-container {
    margin-top: 25vh;
    transform: scale(2);
  }
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
</style>
