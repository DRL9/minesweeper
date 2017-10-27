<template>
    <div class="mine-area" @contextmenu.prevent="1">
        <div class="mine-row" v-for="(col,rIndex) in area" :key="rIndex">
            <div class="mine-col" v-for="(grid,cIndex) in col" :key="cIndex">
                <grid :isActive="grid.isActive" :content="grid.content" @click.native="onGridClick(rIndex,cIndex)" @contextmenu.native="onGridRightClick(rIndex,cIndex)" />
            </div>
        </div>
    </div>
</template>

<script>
import Grid from '@/components/Grid.vue';

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
        Grid
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
            area: []
        };
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
            this.area = this.createMineArea(this.row, this.col, this.mineCount);
        },
        onGridClick (row, col) {
            var grid = this.area[row][col];
            if (grid.content !== '' || grid.isActive) {
                return;
            }
            grid.isActive = true;
            if (grid.hasMine) {
                // game over
                grid.content = 'mine';
            } else if (grid.mineAroundCount === 0) {
                this.spreadEmptyGrid(grid);
            } else {
                grid.content = grid.mineAroundCount;
            }
        },
        onGridRightClick (row, col) {
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
         * @param {Number} row
         * @param {Number} col
         * @param {Number} mineCount
         * @return {Array}
         */
        createMineArea (row, col, mineCount) {
            var coordinates = this.createMineCoordinates(row, col, mineCount);
            var area = this.createEmptyArea(row, col);
            for (let i = 0; i < mineCount; i++) {
                let coor = coordinates[i];
                let [r, c] = coor;
                area[r][c].hasMine = true;
                // 以该坐标为宫格中心，将周围格子的mineAroundCount +1
                for (let grid of genAroundGrids(r, c, area)) {
                    grid.mineAroundCount++;
                }
            }
            return area;
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
                    item.isActive = true;
                    if (item.mineAroundCount === 0) {
                        // 递归激活
                        this.spreadEmptyGrid(item);
                    } else {
                        item.content = item.mineAroundCount;
                    }
                }
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
  width: 40%;
  display: table;
}
.mine-row {
  display: table-row;
}
.mine-col {
  display: table-cell;
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
