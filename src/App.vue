<template>
    <div id="app">
        <mine-area :col="10" :row="10" @gameOver="onGameOver" @gameWin="onGameWin" />
    </div>
</template>

<script>
import MineArea from '@/components/MineArea.vue';
export default {
    components: {
        MineArea
    },
    methods: {
        onGameOver () {

        },
        onGameWin () {

        }
    }
};
</script>
