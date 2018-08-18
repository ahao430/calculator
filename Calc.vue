<template>
  <div class="calculator" :style="{
    display: this.show ? 'block' : 'none',
    left: this.left + 'px',
    bottom: this.bottom + 'px',
  }">
    <div class="calculator-bar" @mousedown="bindStartMove" @mouseup="bindStopMove">
      <span>计算器</span>
      <i class="el-icon-close" @click.stop="hideCalc"></i>
    </div>
    <div class="calculator-wrapper">
      <div class="calculator-header">
        <div class="calculator-result" v-cloak>{{ current }}</div>
      </div>
      <ul class="calculator-main">
        <li v-for="(btn,index) in btns" :key="index" :class="{
          'calculator-item': true,
          'equal': btn === '=',
          'clear': btn === 'C',
        }" @click="operate(btn)">{{btn}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import math from 'mathjs'
math.config({
  number: 'BigNumber',
  precision: 64,
})

export default {
  name: 'calc',
  props: {
    show: {
      type: Boolean,
      default: true,
    },
  },
  data () {
    return {
      btns: [
        'sin', 'cos', 'tan', 'x^', '←', 'C', 'log', 'ln', 'e', '∘', 'rad', '√', '7', '8', '9', '/', 'x²', 'x!', '4', '5', '6', '*', '(', ')', '1', '2', '3', '-', '%', 'π', '0', '.', '±', '+', '=',
      ],
      current: 0,
      startX: 0,
      startY: 0,
      x: 0,
      y: 0,
      startLeft: 0,
      startBottom: 0,
      left: 0,
      bottom: 0,
      moving: false,
      sH: document.documentElement.clientHeight || document.body.clientHeight,
      sW: document.documentElement.clientWidth || document.body.clientWidth,
    }
  },
  computed: {
  },
  mounted () {
    window.math = math
  },
  methods: {
    // 计算
    operate (data) {
      let temp
      if (Number(data) || data === '0') {
        temp = data
        if (this.current === 0) this.current = ''
        this.current += '' + temp
      }
      if (!Number(data)) {
        switch (data) {
        case '/' :
          this.divide()
          break
        case '←' :
          this.backspace()
          break
        case 'C' :
          this.clear()
          break
        case '*' :
          this.multiply()
          break
        case '-' :
          this.minus()
          break
        case '+' :
          this.plus()
          break
        case 'x²' :
          this.square()
          break
        case '(' :
          this.openbracket()
          break
        case ')' :
          this.closebracket()
          break
        case '=' :
          this.equals()
          break
        case '±' :
          this.plusMinus()
          break
        case '%' :
          this.percent()
          break
        case '.' :
          this.decimal()
          break
        case 'sin' :
          this.sin()
          break
        case 'cos' :
          this.cos()
          break
        case 'tan' :
          this.tan()
          break
        case 'ln' :
          this.ln()
          break
        case 'e' :
          this.exp()
          break
        case '∘' :
          this.degrees()
          break
        case 'x!' :
          this.factorial()
          break
        case 'rad' :
          this.radians()
          break
        case '√' :
          this.squareRoot()
          break
        case 'x^' :
          this.exponent()
          break
        case 'π' :
          this.pi()
          break
        case 'log' :
          this.log()
          break
        }
      }
    },
    clear () {
      this.current = 0
    },
    backspace () {
      this.current = this.current.substring(0, this.current.length - 1)
    },
    multiply () {
      this.current += '*'
    },
    divide () {
      this.current += '/'
    },
    openbracket () {
      this.current += '('
    },
    closebracket () {
      this.current += ')'
    },
    plus () {
      this.current += '+'
    },
    minus () {
      this.current += '-'
    },
    decimal () {
      this.current += '.'
    },
    plusMinus () {
      if ((this.current !== 0) && this.current.charAt(0) === '-') {
        this.current = this.current.slice(1)
      } else {
        this.current = '-' + this.current
      }
    },
    equals () {
      this.current = math.number(math.eval(this.current + ''))
    },
    factorial () {
      this.current = math.factorial(this.current)
    },
    pi () {
      this.current = math.number(math.multiply(this.current, math.PI))
    },
    square () {
      this.current = math.square(this.current)
    },
    squareRoot () {
      this.current = math.sqrt(this.current)
    },
    percent () {
      this.current = math.divide(this.current, 100)
    },
    sin () {
      this.current = math.sin(math.multiply(this.current, Math.PI / 180))
    },
    cos () {
      this.current = math.cos(math.multiply(this.current, Math.PI / 180))
    },
    tan () {
      this.current = math.tan(math.multiply(this.current, Math.PI / 180))
    },
    log () {
      this.current = math.log10(this.current)
    },
    ln () {
      this.current = math.log(this.current)
    },
    exponent () {
      this.current += '^'
    },
    exp () {
      this.current = math.exp(this.current)
    },
    radians () {
      this.current = math.number(math.multiply(this.current, Math.PI / 180))
    },
    degrees () {
      this.current = math.number(math.multiply(this.current, 180 / Math.PI))
    },
    // 界面
    hideCalc () {
      this.$emit('close')
    },
    bindStartMove (e) {
      this.startMove(e)
      document.addEventListener('mousemove', this.move)
    },
    bindStopMove (e) {
      this.moving = false
      document.removeEventListener('mousemove', this.move)
    },
    startMove (e) {
      this.startLeft = this.left
      this.startBottom = this.bottom
      this.moving = true
      this.x = e.screenX
      this.startX = e.screenX
      this.y = e.screenY
      this.startY = e.screenY
    },
    move (e) {
      if (this.moving) {
        this.x = e.screenX
        this.y = e.screenY
        this.left = this.startLeft + this.x - this.startX
        this.bottom = this.startBottom + this.startY - this.y
        if (this.left < 0) this.left = 0
        if (this.left > this.sW - 350) this.left = this.sW - 350
        if (this.bottom < 0) this.bottom = 0
        if (this.bottom > this.sH - 452) this.bottom = this.sH - 452
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.calculator{
  border: 1px solid #000;
  padding: 55px 15px 15px;
  width: 350px;
  box-sizing: border-box;
  position: fixed;
  z-index: 3000;
  background: #fff;
  .calculator-bar{
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 40px;
    line-height: 40px;
    display: flex;
    justify-content: space-between;
    box-sizing: border-box;
    padding: 0 10px;
    background: #fff;
    border-bottom: 1px solid #999;
    cursor: move;
    user-select: none;
    i{
      line-height: 40px;
      cursor: pointer;
    }
  }
  .calculator-wrapper{
    width: 320px;
    height: 380px;
    border-radius: 20px;
    box-shadow: 0px 0px  20px 10px #999 inset;
    background: #d2d2bd;
    box-sizing: border-box;
    padding: 20px;
    .calculator-header{
      width: 280px;
      height: 50px;
      border-radius: 10px;
      border: 1px solid #999;
      background: #f7f9f6;
      box-sizing: border-box;
      padding: 10px;
      text-align: right;
      .calculator-result{
        line-height: 30px;
        height: 30px;
        width: 100%;
      }
    }
    .calculator-main{
      display: flex;
      flex-wrap: wrap;
      margin-top: 10px;
      li{
        display: inline-block;
        width: 40px;
        height: 40px;
        line-height: 40px;
        text-align: center;
        box-sizing: border-box;
        border-radius: 6px;
        border: 1px solid #999;
        background: #fff;
        margin: 3px;
        cursor: pointer;
        user-select: none;
        &:active{
          border: 1px solid #999;
          background: darken(#fff, 10%);
        }
        &.clear{
          background: #f00;
          color: #fff;
          &:active{
            background: darken(#f00, 10%)
          }
        }
        &.equal{
          width: 85px;
          background: brown;
          color: #fff;
          font-size: 30px;
          &:active{
            background: darken(brown, 10%)
          }
        }
      }
    }
  }
}
</style>
