<template lang="html">
<div class="wrapper">
    <div class="world">
        <form>
            <label>
                Rows:
                <input type="number" min="1" max="100" v-model="rows" class="form-control">
            </label>
            <label>
                Cols:
                <input type="number" min="1" max="100" v-model="cols" class="form-control">
            </label>
            <label>
                Loops:
                <input type="number" min="1" max="10000" v-model="loops" class="form-control">
            </label>
            <label>
                Timeout (ms between steps):
                <input type="number" min="0" max="1000" v-model="speed" class="form-control">
            </label>
            <button class="btn btn-success" @click.prevent="run" v-if="!running">
                <i class="fas fa-play"></i>
                Run
            </button>
            <button class="btn btn-danger" @click.prevent="cancel" v-if="running">
                <i class="fas fa-times"></i>
                Cancel
            </button>
            <p class="status" v-if="running">
                Loop {{ currentLoop }} / {{ loops }}
            </p>
        </form>
        <table
        tabindex="0"
        @keyup.38="vacuum.row = range(vacuum.row - 1, world.length - 1)"
        @keyup.40="vacuum.row = range(vacuum.row + 1, world.length - 1)"
        @keyup.37="vacuum.col = range(vacuum.col - 1, world[0].length - 1)"
        @keyup.39="vacuum.col = range(vacuum.col + 1, world[0].length - 1)"
        >
            <tbody>
                <tr v-for="(row, i) in world" :key="i">
                    <td v-for="(col, j) in row" :key="j" :class="cellClass(i, j)" @click="dirt(i, j)">
                        <i class="fas fa-recycle"></i>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</div>
</template>

<script>
export default {
    data () {
        return {
            rows: 4,
            cols: 4,
            loops: 1000,
            speed: 100,
            currentLoop: 0,
            running: false,
            timeout: false,
            world: [
                [
                    false, false, false, false
                ], [
                    false, false, false, false
                ], [
                    false, false, false, false
                ], [
                    false, false, false, false
                ]
            ],
            vacuum: {
                row: 0,
                col: 0
            },
            start: {
                row: 0,
                col: 0
            },
            lastWorld: []
        }
    },
    methods: {
        cellClass (row, col) {
            return {
                'vacuum': this.vacuum.row == row && this.vacuum.col == col,
                'trash':  this.world[row][col]
            }
        },
        dirt (i, j) {
            if (this.running) {
                return
            }
            this.world = this.world.map(
                (row, _i) => row.map(
                    (col, _j) => i == _i && j == _j ? !col : col
                )
            )
        },
        updateMap() {
            let world = [];
            for (let j = 0; j < this.rows; j++) {
                world.push([])
                let row = this.world[j]
                for (let i = 0; i < this.cols; i++) {
                    world[j].push(row && row[i])
                }
            }
            this.world = world
        },
        run () {
            this.lastWorld = this.world.map(row => row.map(val => val))
            this.currentLoop = 0
            this.running = true
            this.start.row = this.vacuum.row
            this.start.col = this.vacuum.col
            this.timeout = setInterval(() => {
                this.act()
                this.currentLoop++
                if (this.currentLoop >= this.loops) {
                    this.cancel()
                }
            }, this.speed)
        },
        cancel () {
            clearInterval(this.timeout)
            setTimeout(() => {
                this.restart()
                this.running = false
            }, 5000)
        },
        restart () {
            this.vacuum.row = this.start.row
            this.vacuum.col = this.start.col
            this.world = this.lastWorld.slice()
        },
        act () {
            if (this.shouldClean()) {
                this.clean()
            } else {
                this.move()
            }
        },
        shouldClean () {
            return this.world[this.vacuum.row][this.vacuum.col]
        },
        clean () {
            this.world[this.vacuum.row][this.vacuum.col] = false
        },
        move () {
            if (Math.random() >= 0.5) {
                let row = Math.random() >= 0.5 ? 1 : -1
                this.vacuum.row = this.range(this.vacuum.row + row, this.world.length - 1)
            } else {
                let col = Math.random() >= 0.5 ? 1 : -1
                this.vacuum.col = this.range(this.vacuum.col + col, this.world[0].length - 1)
            }
        },
        range(val, size) {
            return Math.max(0, Math.min(size, val))
        }
    },
    watch: {
        rows (val) {
            this.updateMap()
        },
        cols (val) {
            this.updateMap()
        }
    }
}
</script>

<style lang="css">
.wrapper {
    overflow-x: auto;
    overflow-y: auto;
}
table {
    border: solid black 1px;
    border-collapse: collapse;
}
td {
    border: solid #263238 3px;
    padding: 10px 20px;
    font-size: 200%;
    background-color: #607d8b;
    cursor: pointer;
    color: transparent;
}
td.vacuum {
    background-color: #4caf50;
}
td.trash {
    color: #00bcd4;
}
.status {
    font-weight: bold;
}
</style>
