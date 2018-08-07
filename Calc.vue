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
        <div class="calculator-formula" v-cloak>{{ formula }}</div>
        <div class="calculator-result" v-cloak>{{ result }}</div>
      </div>
      <ul class="calculator-main">
        <li class="calculator-item" @click="operate('7')">7</li>
        <li class="calculator-item" @click="operate('8')">8</li>
        <li class="calculator-item" @click="operate('9')">9</li>
        <li class="calculator-item" @click="drop()">←</li>
        <li class="calculator-item" @click="square()">√</li>
        <li class="calculator-item" @click="operate('4')">4</li>
        <li class="calculator-item" @click="operate('5')">5</li>
        <li class="calculator-item" @click="operate('6')">6</li>
        <li class="calculator-item" @click="operate('+')">+</li>
        <li class="calculator-item" @click="operate('-')">-</li>
        <li class="calculator-item" @click="operate('1')">1</li>
        <li class="calculator-item" @click="operate('2')">2</li>
        <li class="calculator-item" @click="operate('3')">3</li>
        <li class="calculator-item" @click="operate('*')">*</li>
        <li class="calculator-item" @click="operate('/')">/</li>
        <li class="calculator-item" @click="operate('0')">0</li>
        <li class="calculator-item" @click="operate('.')">.</li>
        <li class="calculator-item clear" @click="cleanAll()">C</li>
        <li class="calculator-item equal" @click="equal()">=</li>
      </ul>
    </div>
  </div>
</template>

<script>
import math from 'mathjs' // 引入mathjs, 精确计算
// Math.js works on any ES5 compatible JavaScript engine: node.js 4 or newer, Chrome, Firefox, Safari, Edge, and IE11.
// Though there is no official support for older browsers, math.js should still work on older browsers when using the es5-shim.

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
      formulas: [],
      result: 0,
      editing: false,
      startX: 0,
      startY: 0,
      x: 0,
      y: 0,
      startLeft: 0,
      startBottom: 0,
      left: 0,
      bottom: 0,
      moving: false,
      pattern: /^sqrt\((.+)\)$/,
      sH: document.documentElement.clientHeight || document.body.clientHeight,
      sW: document.documentElement.clientWidth || document.body.clientWidth,
    }
  },
  computed: {
    formula () {
      return this.formulas.join('')
    },
  },

  methods: {
    operate (element) {
      if (this.editing) {
        let last = this.formulas[this.formulas.length - 1]
        switch (element) {
        case '0':
        case '1':
        case '2':
        case '3':
        case '4':
        case '5':
        case '6':
        case '7':
        case '8':
        case '9':
          if (last === '.') {
            this.formulas.pop()
            this.formulas.push(parseFloat('.' + element))
          } else if (!Number.isNaN(+last)) {
            this.formulas.pop()
            this.formulas.push(last + '' + element)
          } else if (this.pattern.test(last)) {
            // do nothing
          } else {
            this.formulas.push(element)
          }
          break
        case '.':
          if (last.indexOf('.') > -1) {
            // do nothing
          } else if (!Number.isNaN(+last)) {
            this.formulas.pop()
            this.formulas.push(last + '' + element)
          } else {
            this.formulas.push(element)
          }
          break
        case '+':
        case '-':
        case '*':
        case '/':
        default:
          if (last === '.') {
            this.formulas.pop()
            this.formulas.push(0)
            this.formulas.push(element)
          } else if (!Number.isNaN(+last)) {
            this.formulas.push(element)
          } else {
            let matches = last.match(this.pattern)
            if (matches) {
              this.formulas.push(element)
            } else {
              this.formulas.pop()
              this.formulas.push(element)
            }
          }
        }
      } else {
        switch (element) {
        case '0':
        case '1':
        case '2':
        case '3':
        case '4':
        case '5':
        case '6':
        case '7':
        case '8':
        case '9':
        case '.':
          this.formulas = [element]
          break
        case '+':
        case '-':
        case '*':
        case '/':
        default:
          this.formulas = [this.result, element]
        }
        this.editing = true
      }
    },

    equal () {
      try {
        this.result = math.parser().eval(this.formula) || 0
      } catch (err) {
        console.log(err)
        this.result = '错误'
      }
      this.editing = false
    },

    cleanAll () {
      this.formulas = []
      this.result = 0
      this.editing = false
    },

    drop () {
      let last = this.formulas[this.formulas.length - 1]
      if (!Number.isNaN(+last)) {
        this.formulas.pop()
        if (last.length > 1) {
          this.formulas.push(last.slice(0, last.length - 1))
        }
      } else {
        let matches = last.match(this.pattern)
        if (matches) {
          this.formulas.pop()
          this.formulas.push(matches[1])
        } else {
          this.formulas.pop()
        }
        if (this.formulas.length === 0) {
          this.editing = false
        }
      }
    },

    square () {
      if (this.editing) {
        let last = this.formulas[this.formulas.length - 1]
        if (!Number.isNaN(+last)) {
          this.formulas.pop()
          this.formulas.push('sqrt(' + last + ')')
        } else if (last === '.') {
          this.formulas.pop()
          this.formulas.push('sqrt(0)')
        } else {
          let matches = last.match(this.pattern)
          if (matches) {
            this.formulas.pop()
            this.formulas.push('sqrt(' + last + ')')
          }
        }
      } else {
        this.formulas = ['sqrt(' + this.result + ')']
        this.equal()
      }
    },

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
        if (this.left > this.sW - 300) this.left = this.sW - 300
        if (this.bottom < 0) this.bottom = 0
        if (this.bottom > this.sH - 352) this.bottom = this.sH - 352
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.calculator{
  border: 1px solid #000;
  padding: 55px 15px 15px;
  width: 300px;
  box-sizing: border-box;
  position: fixed;
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
    width: 270px;
    height: 280px;
    border-radius: 20px;
    box-shadow: 0px 0px  20px 10px #999 inset;
    background: #d2d2bd;
    box-sizing: border-box;
    padding: 20px;
    .calculator-header{
      width: 230px;
      height: 60px;
      border-radius: 10px;
      border: 1px solid #999;
      background: #f7f9f6;
      box-sizing: border-box;
      padding: 10px;
      text-align: right;
      .calculator-formula{
        height: 26px;
        line-height: 12px;
        width: 100%;
        user-select: none;
        font-size: 12px;
        word-break: break-all;
        overflow : hidden;
        text-overflow: ellipsis;
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        color: #333;
      }
      .calculator-result{
        line-height: 15px;
        height: 15px;
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
