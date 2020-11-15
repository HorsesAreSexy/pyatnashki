<template>
  <div class="box">
    <div class="field">
      <div class="top side">
        <div class="title">
          Pyatnashki
        </div>
        <img class="fire" src="fire.gif">
      </div>
      <div class="left side steps">
        ходов
        <div class="count">
          {{ count }}
        </div>
      </div>
      <div class="bottom side time">
        {{ time }}
      </div>
      <Number v-for="structObject in structure" :key="structObject.number" :data="structObject" @slide="onSlide" />
    </div>
    <div v-show="won" id="animation" class="winscreen">
      <div class="message">
        <div class="text">
          U spend <b>><span class="time">{{ finalTime }}</span>&#60;</b> minutes of ur life in this old sptupid game.
          <div class="congartz">
            Congratulations!
          </div>
        </div>
        <img class="celestia" src="clap.gif">
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import randomInt from 'random-int'
export default {
  data () {
    let length = 16
    const numbers = Array(length).fill(null).map((_, index) => index + 1)
    const structure = []
    while (length) {
      const randomIndex = randomInt(1, length--) - 1
      const randomValue = +numbers.splice(randomIndex, 1)
      const structObject = Vue.observable({
        number: randomValue === 16 ? null : randomValue,
        slide: null,
        dragging: null
      })
      structure.push(structObject)
    }
    return {
      structure,
      count: 0,
      time: '00:00',
      finalTime: 0,
      won: false
    }
  },
  mounted () {
    this.onSlide()
    this.ticking = setInterval(this.tick, 1000)
  },
  methods: {
    win () {
      clearInterval(this.ticking)
      this.finalTime = Number(this.time.slice(0, 2))
      this.won = true
      const winScreen = document.getElementById('animation')
      const duration = 4e2
      winScreen.animate([
        { opacity: 0 },
        { opacity: 1 }
      ], { duration })
      winScreen.firstChild.animate([
        { top: '100px' },
        { top: '0px' }
      ], { duration })
    },
    tick () {
      const restrict = [9, 5, null, 9]
      const perDigit = this.time.split('').reverse()
      for (const index in perDigit) {
        const maxNumber = restrict[index]
        if (maxNumber === null) {
          continue
        }
        const currentNumber = +perDigit[index]
        if (currentNumber === maxNumber) {
          perDigit[index] = 0
        } else {
          perDigit[index] = currentNumber + 1
          break
        }
      }
      this.time = perDigit.reverse().join('')
    },
    onSlide (newVoid) {
      const { structure } = this
      if (newVoid) {
        this.count += 1
        const oldVoid = structure.find(({ number }) => number === null)
        oldVoid.number = newVoid.number
        newVoid.number = null
        let check = 0
        if (structure.slice(0, -1).every(({ number }) => ++check === number)) {
          this.win()
        }
      }
      for (const structObject of structure) {
        if (structObject.slide) {
          structObject.slide = null
        }
      }
      const position = structure.findIndex(({ number }) => number === null) + 1
      const shiftingMap = new Map([
        [
          [1],
          [1, 4]
        ],
        [
          [4],
          [-1, 4]
        ],
        [
          [13],
          [1, -4]
        ],
        [
          [16],
          [-1, -4]
        ],
        [
          [2, 3],
          [-1, 1, 4]
        ],
        [
          [5, 9],
          [-4, 4, 1]
        ],
        [
          [8, 12],
          [-4, 4, -1]
        ],
        [
          [14, 15],
          [-4, 1, -1]
        ],
        [
          [6, 7, 10, 11],
          [-4, 4, -1, 1]
        ]
      ])
      for (const [match, shifts] of shiftingMap) {
        if (match.includes(position)) {
          for (const shift of shifts) {
            structure[position + shift - 1].slide = {
              '-4': 'bottom',
              '-1': 'right',
              1: 'left',
              4: 'top'
            }[shift]
          }
          break
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.box{
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
}
.field{
  outline: 60px solid black;
  width: 600px;
  height: 600px;
  margin: auto;
  position: relative;
  .side{
    position: absolute;
    text-shadow: 2px 2px 0 white, -2px -2px 0 white;
    &.top{
      bottom: 100%;
    }
    &.left{
      right: 100%;
    }
    &.bottom{
      top: 100%;
    }
  }
  .title{
    text-align: center;
    font-size: 90px;
    font-weight: bold;
    position: absolute;
    width: 100%;
    bottom: 10px;
    text-transform: uppercase;
  }
  .fire{
    position: relative;
    bottom: -7px;
  }
  .steps, .time{
    text-transform: uppercase;
    font-size: 120px;
    font-weight: bold;
  }
  .steps{
    writing-mode: vertical-rl;
    margin-right: -20px;
    .count{
      height: 140px;
      text-align: center;
      display: inline-block;
    }
  }
  .time{
    margin-top: -20px;
    text-align: center;
    width: 100%;
  }
}
.winscreen{
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.7);
  display: flex;
  z-index: 999;
  .message{
    margin: auto;
    outline: 30px solid black;
    width: 80%;
    height: 300px;
    position: relative;
    background: white;
    display: flex;
    justify-content: space-between;
    .text{
      padding: 15px 30px;
      font-size: 28px;
      .time{
        font-size: 36px;
      }
      .congartz{
        font-size: 48px;
        font-weight: bold;
        line-height: 2;
      }
    }
    .celestia{
      height: 100%;
      float: right;
      border-left: 4px solid black;
    }
  }
}
</style>
