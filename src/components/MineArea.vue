<template>
    <div class="mine-area" @contextmenu.prevent="1">
        <div class="mine-row" v-for="(col,rIndex) in area" :key="rIndex">
            <div class="mine-col" v-for="(grid,cIndex) in col" :key="cIndex">
                <mine-grid :isActive="grid.isActive" :content="grid.content" @click.native="onGridClick(rIndex,cIndex)" @contextmenu.native="onGridRightClick(rIndex,cIndex)" />
            </div>
        </div>
    </div>
</template>

<script>
import MineGrid from '@/components/MineGrid.vue';

function * genAroundGrids (r, c, area) {
    var row = area.length;
    var col = area[0].length;
    for (let a = -1; a <= 1; a++) {
        for (let b = -1; b <= 1; b++) {
            let x = r + a;
            let y = c + b;
            let isOutOfBound = (x < 0 ||
                    x >= row ||
                    y < 0 ||
                    y >= col);
            if (!isOutOfBound) {
                yield area[x][y];
            }
        }
    }
}

export default {
    components: {
        MineGrid
    },
    props: {
        col: {
            type: Number,
            default: 9,
            validator (val) {
                return val > 1 && val <= 20;
            }
        },
        row: {
            type: Number,
            default: 9,
            validator (val) {
                return val > 1 && val <= 20;
            }
        },
        mineCount: {
            type: Number,
            default: 10
        }
    },
    data () {
        return {
            area: [],
            mineCoordinates: [],
            gameRunning: false,
            gridCountActive: 0
        };
    },
    computed: {
        gridCountTotal () {
            return this.col * this.row;
        }
    },
    created () {
        this.setArea();
    },
    watch: {
        col () {
            this.setArea();
        },
        row () {
            this.setArea();
        },
        mineCount () {
            this.setArea();
        }
    },
    methods: {
        setArea () {
            this.area = this.createEmptyArea(this.row, this.col);
            this.gameRunning = true;
            this.Mine(this.row, this.col, this.mineCount, this.area);
        },
        onGridClick (row, col) {
            if (!this.gameRunning) return;

            var grid = this.area[row][col];
            if (grid.content !== '' || grid.isActive) {
                return;
            }
            this.activeGrid(grid);
            if (grid.hasMine) {
                // game over
                grid.content = 'mine';
                this.handleGameOver();
                return;
            }
            if (grid.mineAroundCount === 0) {
                this.spreadEmptyGrid(grid);
            } else {
                grid.content = grid.mineAroundCount;
            }
            this.detectWin();
        },
        onGridRightClick (row, col) {
            if (!this.gameRunning) return;

            let grid = this.area[row][col];
            if (grid.isActive) return;
            switch (grid.content) {
            case '':
                grid.content = 'flag';
                break;
            case 'flag':
                grid.content = 'questionmark';
                break;
            default:
                grid.content = '';
                break;
            }
        },
        activeGrid (grid) {
            grid.isActive = true;
            this.gridCountActive++;
        },
        /**
         * @param {Number} row
         * @param {Number} col
         * @return {Array}
         */
        createEmptyArea (row, col) {
            var area = [];
            for (let i = 0; i < row; i++) {
                let grids = [];
                for (let j = 0; j < col; j++) {
                    grids.push({
                        isActive: false,
                        content: '',
                        mineAroundCount: 0,
                        hasMine: false,
                        r: i,
                        c: j
                    });
                }
                area.push(grids);
            }
            return area;
        },
        /**
         * @param {Number} row
         * @param {Number} col
         * @param {Number} mineCount
         * @return {Array}
         */
        createMineCoordinates (row, col, mineCount) {
            var coors = [];
            var result = [];
            for (let i = 0; i < row; i++) {
                for (let j = 0; j < col; j++) {
                    coors.push(`${i},${j}`);
                }
            }
            for (let i = 0; i < mineCount; i++) {
                var index = Math.floor(Math.random() * coors.length);
                var val = coors.splice(index, 1)[0];
                result.push(val.split(',').map(Number));
            }
            return result;
        },
        /**
         * 布雷
         * @param {Number} row
         * @param {Number} col
         * @param {Number} mineCount
         * @param {Array} area
         */
        Mine (row, col, mineCount, area) {
            var coordinates = this.mineCoordinates = this.createMineCoordinates(row, col, mineCount);
            for (let i = 0; i < mineCount; i++) {
                let coor = coordinates[i];
                let [r, c] = coor;
                area[r][c].hasMine = true;
                // 以该坐标为宫格中心，将周围格子的mineAroundCount +1
                for (let grid of genAroundGrids(r, c, area)) {
                    grid.mineAroundCount++;
                }
            }
        },
        /**
         * 如果一个格子没有雷，且周围的雷数为0，那么激活它周围的未激活和未标志的格子
         * @param {Object} grid
         *
         */
        spreadEmptyGrid (grid) {
            var aroundGridsIterator = genAroundGrids(grid.r, grid.c, this.area);
            for (let item of aroundGridsIterator) {
                if (!item.isActive && item.content === '') {
                    // 该格子未激活，未被标识
                    this.activeGrid(item);
                    if (item.mineAroundCount === 0) {
                        // 递归激活
                        this.spreadEmptyGrid(item);
                    } else {
                        item.content = item.mineAroundCount;
                    }
                }
            }
        },
        handleGameOver () {
            this.gameRunning = false;
            for (let coor of this.mineCoordinates) {
                let [x, y] = coor;
                let grid = this.area[x][y];
                grid.content = 'mine';
                grid.isActive = true;
            }
            this.$emit('gameOver');
        },
        detectWin () {
            if (this.gridCountTotal === this.gridCountActive + this.mineCount) {
                this.$emit('gameWin');
                this.gameRunning = false;
            }
        }
    }
};
</script>

<style scope>
.mine-area {
  --grid-spacing: 3%;
  --grid-spacing-keydown: 6%;
  margin: auto;
  max-width: 1000px;
  display: table;
}
.mine-row {
  display: table-row;
}
.mine-col {
  display: table-cell;
  width: 50px;
  min-width: 25px;
  position: relative;
}
.mine-col::before {
  content: "";
  display: block;
  padding-top: 100%;
}
.mine-col > * {
  position: absolute;
  top: var(--grid-spacing);
  left: var(--grid-spacing);
  bottom: var(--grid-spacing);
  right: var(--grid-spacing);
  transition: all 0.1s ease-out;
}

.mine-col > *:active {
  top: var(--grid-spacing-keydown);
  left: var(--grid-spacing-keydown);
  bottom: var(--grid-spacing-keydown);
  right: var(--grid-spacing-keydown);
}
</style>
