<template>
  <div id="app">
    <header>
      <div class="statsbox">
        <ul>
          <li>Remaining: {{ remaining() }}</li>
          <li ref="simplesinglehint" @click="showHint">Simple Singles: {{ this.hints.filter(x => x.type === 'Simple Single').length }} {{ hintDetail != null ? ' (' + hintDetail + ')' : '' }}</li>
        </ul>
      </div>
    </header>
    <main>
      <div class="playbox">
        <table>
          <tr v-for="(row, rowIx) in activeGrid" :key="rowIx">
            <td v-for="(cell, colIx) in activeGrid[rowIx]" :key="colIx"
              :class="classGrid[rowIx][colIx]"
              @click="select(rowIx, colIx)"
            >
              {{ cell }}
            </td>
          </tr>
        </table>
      </div>
    </main>
    <footer>
      <ul class="numbers">
        <li v-for="num in [1, 2, 3, 4, 5, 6, 7, 8, 9]" :key="num">
          <p @click="insert(num)">{{ num }}</p>
        </li>
      </ul>
    </footer>
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
      possibilitiesGrid: [
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
        [[], [], [], [], [], [], [], [], []],
      ],
      conflict: { result: 'good' },
      selectY: null,
      selectX: null,
      hints: [],
      displayHint: false,
      hintDetail: null
    }
  },
  mounted() {
    this.refreshPossibilitiesGrid()
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
      if (this.displayHint && this.hints.length > 0) {
        const hint = this.hints[0]
        classes[hint.y][hint.x] = 'highlight'
      }
      if (this.selectY != null && this.selectX != null) {
        classes[this.selectY][this.selectX] = 'selected'
      }
      if (this.conflict.result === 'good') return classes
      if (this.conflict.type === 'row') {
        for (let i = 0; i < 9; i++) classes[this.conflict.y][i] = 'highlight'
        classes[this.conflict.y][this.conflict.x[0]] += ' conflict'
        classes[this.conflict.y][this.conflict.x[1]] += ' conflict'
      }
      if (this.conflict.type === 'col') {
        for (let i = 0; i < 9; i++) classes[i][this.conflict.x] = 'highlight'
        classes[this.conflict.y[0]][this.conflict.x] += ' conflict'
        classes[this.conflict.y[1]][this.conflict.x] += ' conflict'
      }
      if (this.conflict.type === 'square') {
        const baseX = (this.conflict.square % 3) * 3
        const baseY = parseInt(this.conflict.square / 3) * 3
        for (let i = 0; i < 3; i++) {
          for (let j = 0; j < 3; j++) {
            classes[baseY + i][baseX + j] = 'highlight'
          }
        }
        classes[this.conflict.coords[0].y][this.conflict.coords[0].x] += ' conflict'
        classes[this.conflict.coords[1].y][this.conflict.coords[1].x] += ' conflict'
      }
      return classes
    }
  },
  methods: {
    remaining() {
      return this.activeGrid.flat().filter(x => x == null).length
    },
    select(y, x) {
      this.selectY = y
      this.selectX = x
    },
    insert(num) {
      if (this.selectY == null || this.selectX == null) return
      this.activeGrid[this.selectY][this.selectX] = parseInt(num)
      this.selectY = null
      this.selectX = null
      this.conflict = this.verifyGrid(this.activeGrid)
      this.refreshPossibilitiesGrid()
      this.displayHint = false
      this.hintDetail = null
      // this.$forceUpdate()
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
            square: squareIndex,
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
    },
    refreshPossibilitiesGrid() {
      this.hints = []
      const active = this.activeGrid.slice()
      for (let i = 0; i < 9; i++) {
        for (let j = 0; j < 9; j++) {
          if (active[i][j] != null) continue
          const rowPoss = this.getPossibilities(active[i].slice())
          const colPoss = this.getPossibilities(active.map(x => x[j]))
          const sqPoss = this.getPossibilities(this.pullSquare(this.gridToSquare({ y: i, x: j }).square).flat())
          const poss = rowPoss.filter(x => colPoss.includes(x)).filter(y => sqPoss.includes(y))
          this.possibilitiesGrid[i][j] = poss
          if (poss.length === 1) {
            this.hints.push({
              y: i,
              x: j,
              type: 'Simple Single',
              number: poss[0]
            })
          }
        }
      }
    },
    getPossibilities(list) {
      const uniq = this.sortUniq(list)
      return [1, 2, 3, 4, 5, 6, 7, 8, 9].filter(x => !uniq.includes(x))
    },
    sortUniq(arr) {
      if (arr.length === 0) return arr;
      arr = arr.sort(function (a, b) { return a*1 - b*1; });
      var ret = [arr[0]];
      for (var i = 1; i < arr.length; i++) {
        if (arr[i-1] !== arr[i]) {
          ret.push(arr[i]);
        }
      }
      return ret;
    },
    showHint() {
      if (this.displayHint) this.hintDetail = this.hints[0].number
      this.displayHint = true
    }
  }
}
</script>

<style>
header{
  background-color: lightskyblue;
}
.statsbox {
    padding: 20px 10px;
    line-height: 20px;
}
table {
  width: 100%;
  height: 100%;
}
td {
  border: 1px solid black;
  font-family: monospace;
  font-size: 100%;
  font-weight: 600;
  text-align: center;
}
input {
  width: 100%;
  height: 100%;
  font-family: monospace;
  font-size: 100%;
  font-weight: 600;
  text-align: center;
}
.highlight {
  border-color: pink;
}
.normal {
  background-color: white;
}
.conflict {
  background-color: red;
}
.selected {
  background-color: green;
}
.playbox {
  max-width: 350px;
  height: 350px;
  margin: auto;
}
.numbers > li {
  float: left;
  width: 11%
}
.stats > li {
  float: left;
}
main {
  min-height: calc(100vh - 190px);
  margin-top: 50px;
}
footer {
  height: 80px;
  width: 90%;
}
ul {
  list-style-type: none;
}
li > p {
  text-align: center;
  display: block;
  padding: 5px 5px;
  font-size: 18px;
  border: 1px solid gray;
  border-radius: 10px;
  margin: 2px;
}
.numbers :hover {
  border-color: lightskyblue;
  color: rgb(10, 145, 230);
}
</style>
