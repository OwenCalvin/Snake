<template>
  <div>
    <div class="game-content">
      <div class="game-area">
        <div class="game-x" v-for="x in game.width" :key="'x' + x">
          <div class="game-y" v-for="y in game.height" :key="'y' + y">
            <div class="case" :data-x="x" :data-y="y"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
function addVectors (v1, v2) {
  return {
    x: v1.x + v2.x,
    y: v1.y + v2.y
  }
}

function multiplyVector (coefficient, vector) {
  return {
    x: vector.x * coefficient,
    y: vector.y * coefficient
  }
}

function intersect (p1, p2) {
  return p1.x === p2.x && p1.y === p2.y
}

function randomPoint (width, height) {
  return {
    x: Math.floor((Math.random() * width) + 1),
    y: Math.floor((Math.random() * height) + 1)
  }
}

function getCase (x, y) {
  return document.querySelector(`[data-x='${x}'][data-y='${y}']`)
}

function copyObject (obj) {
  return JSON.parse(JSON.stringify(obj))
}

function setColor (c, i = null) {
  c.style.backgroundColor = i ? `rgba(50, 50, 50, ${(0.65 / i) + 0.3})` : ''
}

const initialTime = 150
const minTime = 80

const initialSnake = {
  points: [
    {
      pos: {
        x: 1,
        y: 1
      },
      v: {
        x: 1,
        y: 0
      }
    }
  ],
  cases: []
}

export default {
  name: 'snake',
  data () {
    // 16:9
    let width = 32
    let height = 18
    return {
      game: {
        time: initialTime,
        lose: false,
        width: width,
        height: height
      },
      snake: copyObject(initialSnake),
      point: {
        pos: randomPoint(width, height),
        case: null
      },
      breaks: []
    }
  },
  computed: {
    snakeHead () {
      return this.snake.points[0]
    },
    snakeBody () {
      return this.snake.points[1] || { pos: addVectors(this.snakeHead.pos, multiplyVector(-1, this.snakeHead.v)) }
    },
    snakeAss () {
      return this.snake.points[this.snake.points.length - 1]
    },
    canMoveY () {
      return this.snakeHead.pos.y === this.snakeBody.pos.y
    },
    canMoveX () {
      return this.snakeHead.pos.x === this.snakeBody.pos.x
    }
  },
  methods: {
    getBreak (v) {
      return {
        pos: this.snakeHead.pos,
        v: v,
        count: []
      }
    },
    resetGame () {
      this.snake.cases.forEach(c => { setColor(c) })
      this.game.time = initialTime
      this.snake = copyObject(initialSnake)
      this.breaks = []
    }
  },
  mounted () {
    this.point.case = getCase(this.point.pos.x, this.point.pos.y)
    this.point.case.classList.add('point')

    window.addEventListener('keydown', e => {
      if ([37, 38, 39, 40].indexOf(e.keyCode) > -1) {
        e.preventDefault()
      }
      switch (e.keyCode) {
        // Up
        case 38:
          if (this.canMoveY) {
            this.breaks.push(this.getBreak({ x: 0, y: -1 }))
          }
          break
        // Down
        case 40:
          if (this.canMoveY) {
            this.breaks.push(this.getBreak({ x: 0, y: 1 }))
          }
          break
        // Left
        case 37:
          if (this.canMoveX) {
            this.breaks.push(this.getBreak({ x: -1, y: 0 }))
          }
          break
        // Right
        case 39:
          if (this.canMoveX) {
            this.breaks.push(this.getBreak({ x: 1, y: 0 }))
          }
          break
      }
    })

    const run = () => {
      setTimeout(game, this.game.time)
    }

    const game = () => {
      // Reset snake
      this.snake.cases.forEach(c => { setColor(c) })
      this.snake.cases = []

      for (let [iP, p] of this.snake.points.entries()) {
        for (let [iB, b] of this.breaks.entries()) {
          // Intersect a breakpoint
          if (intersect(p.pos, b.pos)) {
            if (!b.count.includes(iP)) {
              b.count.push(iP)
              p.v = b.v
            }
            // Remove breakpoint if all snake points walked on it
            if (b.count.length >= this.snake.points.length) {
              this.breaks.splice(iB, 1)
            }
          }
        }

        // Walk
        p.pos = addVectors(p.pos, p.v)
        let pCase = getCase(p.pos.x, p.pos.y)
        if (pCase) {
          setColor(pCase, iP + 1)
          this.snake.cases.push(pCase)
        } else {
          // Touch corner
          this.resetGame()
        }
      }

      // If snake eat
      if (intersect(this.snakeHead.pos, this.point.pos)) {
        // New snake point
        let newSnakePoint = addVectors(this.snakeAss.pos, multiplyVector(-1, this.snakeAss.v))
        this.snake.points.push({
          pos: { ...newSnakePoint },
          v: this.snakeAss.v
        })

        // New random point
        this.point.case.classList.remove('point')
        let newPoint = randomPoint(this.game.width, this.game.height)
        let newCase = getCase(newPoint.x, newPoint.y)
        newCase.classList.add('point')
        this.point = {
          pos: newPoint,
          case: newCase
        }

        this.game.time -= this.game.time > minTime ? 5 : 0
      }

      // If snake intersect himself
      for (let [iP1, p1] of this.snake.points.entries()) {
        for (let [iP2, p2] of this.snake.points.entries()) {
          if (iP1 !== iP2 && intersect(p1.pos, p2.pos)) {
            this.resetGame()
          }
        }
      }
      run()
    }
    run()
  }
}
</script>

<style lang="scss" scoped>

.point {
  position: relative;
  &:after {
    content: '';
    opacity: .5;
    position: absolute;
    top: 0;
    left: 0;
    background: rgba(50, 50, 50, .5);
    border-radius: 30px;
    width: 100%;
    height: 100%;
    animation: bye 2s infinite;
  }
  &:before {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    background: rgb(50, 50, 50);
    border-radius: 30px;
    width: 65%;
    height: 65%;
    transform: translateX(-50%) translateY(-50%);
  }
}

.game-content {
  display: flex;
  justify-content: center;
}

.game-area {
  display: flex;
  background: rgb(255, 255, 255);
  justify-content: center;
  flex-wrap: nowrap;
  border-radius: 7px;
  overflow: hidden;
  box-shadow: 0px 13px 57px -10px rgba(0, 0, 0, 0.1);
}

.case {
  width: 30px;
  height: 30px;
}

@keyframes bye {
    from {
      transform: scale(0);
      opacity: 1;
    }
    50% {
      transform: scale(1.25);
    }
    to {
      transform: scale(1.5);
      opacity: 0;
    }
}

@media (max-width: 1000px) {
  .case {
    width: 20px;
    height: 20px;
  }
}

@media (max-width: 750px) {
  .case {
    width: 15px;
    height: 15px;
  }
}

@media (max-width: 600px) {
  .case {
    width: 13px;
    height: 13px;
  }
}

@media (max-width: 450px) {
  .case {
    width: 11px;
    height: 11px;
  }
}
</style>
