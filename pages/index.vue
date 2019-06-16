<template>
  <div class="playbox">
    <table>
      <tr v-for="(row, rowIx) in activeGrid" :key="rowIx">
        <td v-for="(cell, colIx) in activeGrid[rowIx]" :key="colIx"
          :class="classGrid[rowIx][colIx]"
        >
          <input
            type="number"
            :disabled="cell > 0 && cell < 10 && classGrid[rowIx][colIx] === 'normal'"
            v-model="activeGrid[rowIx][colIx]"
            size="1"
            min="1"
            max="9"
            @change="validate(rowIx, colIx)"
          >
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      solution: [
        [5, 3, 4, 6, 7, 8, 9, 1, 2],
        [6, 7, 2, 1, 9, 5, 3, 4, 8],
        [1, 9, 8, 3, 4, 2, 5, 6, 7],
        [8, 5, 9, 7, 6, 1, 4, 2, 3],
        [4, 2, 6, 8, 5, 3, 7, 9, 1],
        [7, 1, 3, 9, 2, 4, 8, 5, 6],
        [9, 6, 1, 5, 3, 7, 2, 8, 4],
        [2, 8, 7, 4, 1, 9, 6, 3, 5],
        [3, 4, 5, 2, 8, 6, 1, 7, 9],
      ],
      activeGrid: [
        [5, 3, null, null, 7, null, null, null, null],
        [6, null, null, 1, 9, 5, null, null, null],
        [null, 9, 8, null, null, null, null, 6, null],
        [8, null, null, null, 6, null, null, null, 3],
        [4, null, null, 8, null, 3, null, null, 1],
        [7, null, null, null, 2, null, null, null, 6],
        [null, 6, null, null, null, null, 2, 8, null],
        [null, null, null, 4, 1, 9, null, null, 5],
        [null, null, null, null, 8, null, null, 7, 9],
      ],
      conflict: { result: 'good' }
    }
  },
  computed: {
    classGrid() {
      let classes = [
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
        ['normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', 'normal', ],
      ]
      console.log(this.conflict)
      if (this.conflict.result === 'good') return classes
      if (this.conflict.type === 'row') {
        classes[this.conflict.y][this.conflict.x[0]] = 'conflict'
        classes[this.conflict.y][this.conflict.x[1]] = 'conflict'
      }
      if (this.conflict.type === 'col') {
        classes[this.conflict.y[0]][this.conflict.x] = 'conflict'
        classes[this.conflict.y[1]][this.conflict.x] = 'conflict'
      }
      if (this.conflict.type === 'square') {
        classes[this.conflict.coords[0].y][this.conflict.coords[0].x] = 'conflict'
        classes[this.conflict.coords[1].y][this.conflict.coords[1].x] = 'conflict'
      }
      return classes
    }
  },
  methods: {
    validate(y, x) {
      const num = parseInt(this.activeGrid[y][x])
      if (num > 0 && num < 10) {
        this.activeGrid[y][x] = num
        this.conflict = this.verifyGrid(this.activeGrid)
      } else {
        this.activeGrid[y][x] = null
      }
    },
    verifyGrid(grid) {
      let conf = {}
      for (let i = 0; i < 9; i++) {
        conf = this.verifyRow(grid[i], i)
        if (conf.result !== 'good') return conf
      }
      if (conf.result === 'good') {
        for (let i = 0; i < 9; i++) {
          conf = this.verifyCol(grid.map(x => x[i]), i)
          if (conf.result !== 'good') return conf
        }
      }
      if (conf.result === 'good') {
        for (let i = 0; i < 9; i++) {
          conf = this.verifySquare(this.pullSquare(i), i)
          if (conf.result !== 'good') return conf
        }
      }
      return conf
    },
    verifyCol(line, colIndex) {
      const result = this.verifyLine(line)
      if (result.result === 'good') return result
      return { result: 'conflict', type: 'col', number: result.number, y: result.indexes, x: colIndex }
    },
    verifyRow(line, rowIndex) {
      const result = this.verifyLine(line)
      if (result.result === 'good') return result
      return { result: 'conflict', type: 'row', number: result.number, y: rowIndex, x: result.indexes }
    },
    verifyLine(line) {
      const myline = line.slice()
      for (let i = 1; i <= 9; i++) {
        if (myline.filter(x => x === i).length > 1) {
          const first = myline.findIndex(x => x === i)
          myline.splice(0, first + 1)
          const second = myline.findIndex(x => x === i)
          return { result: 'conflict', number: i, indexes: [ first, second + first + 1 ] }
        }
      }
      return { result: 'good' }
    },
    verifySquare(list, squareIndex) {
      const mysquare = list.slice().flat()
      for (let i = 1; i <= 9; i++) {
        if (mysquare.filter(x => x === i).length > 1) {
          const first = mysquare.findIndex(x => x === i)
          mysquare.splice(0, first + 1)
          const second = mysquare.findIndex(x => x === i)
          return {
            result: 'conflict',
            type: 'square',
            number: i,
            coords: [
              this.squareToGrid({
                square: squareIndex,
                x: first % 3,
                y: parseInt(first / 3)
              }),
              this.squareToGrid({
                square: squareIndex,
                x: (second + first + 1) % 3,
                y: parseInt((second + first + 1) / 3)
              }),
            ]
          }
        }
      }
      return { result: 'good' }
    },
    pullSquare(squareIndex) {
      let baseX = (squareIndex % 3) * 3
      let baseY = Math.floor(squareIndex / 3) * 3
      return [
        [this.activeGrid[baseY][baseX], this.activeGrid[baseY][baseX + 1], this.activeGrid[baseY][baseX + 2] ],
        [this.activeGrid[baseY + 1][baseX], this.activeGrid[baseY + 1][baseX + 1], this.activeGrid[baseY + 1][baseX + 2] ],
        [this.activeGrid[baseY + 2][baseX], this.activeGrid[baseY + 2][baseX + 1], this.activeGrid[baseY + 2][baseX + 2] ],
      ]
    },
    squareToGrid(squareNotation) {
      let baseX = (squareNotation.square % 3) * 3
      let baseY = Math.floor(squareNotation.square / 3) * 3
      return { x: baseX + squareNotation.x, y: baseY + squareNotation.y }
    },
    gridToSquare(GridNotation) {
      return {
        square: Math.floor(GridNotation.x / 3) + Math.floor(GridNotation.y / 3) * 3,
        x: GridNotation.x % 3,
        y: GridNotation.y % 3,
      }
    }
  }
}
</script>

<style>
table {
  width: 100%;
  height: 100%;
}
td {
  border: 1px solid black;
}
input {
  width: 30px;
  font-family: monospace;
  font-size: 20px;
  text-align: center;
}
.normal {
  background-color: white;
}
.conflict {
  background-color: red
}
.playbox {
  padding-top: 35px;
  width: 350px;
  height: 350px;
  margin: 0 auto;
}
</style>
