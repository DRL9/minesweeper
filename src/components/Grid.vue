<template>
    <div :class="isActive?'active':''">
        <flag v-if="showFlag" />
        <mine v-if="showMine" />
        <question-mark v-if="showQuestionMark" :color="'#FFCB10'" />
        <digit v-if="mineAroundCount>0" :number="mineAroundCount" />
    </div>
</template>

<script>
import Flag from '@/components/BaseFlag.vue';
import Mine from '@/components/BaseMine.vue';
import QuestionMark from '@/components/BaseQuestionMark.vue';
import Digit from '@/components/BaseDigit.vue';

export default {
    components: {
        Flag,
        Mine,
        QuestionMark,
        Digit
    },
    props: {
        isActive: {
            type: Boolean,
            required: true
        },
        content: {
            type: String | Number,
            required: true,
            validator (val) {
                const icon = [ 'flag', 'mine', 'questionmark', '' ];
                return (val > 0 && val < 9) || icon.indexOf(val) > -1;
            }
        }
    },
    data () {
        return {
            showFlag: false,
            showQuestionMark: false,
            showMine: false,
            mineAroundCount: -1
        };
    },
    watch: {
        content (val) {
            this.showFlag = val === 'flag';
            this.showQuestionMark = val === 'questionmark';
            this.showMine = val === 'mine';
            if (typeof val === 'number' && val > 0 && val < 9) {
                this.mineAroundCount = val;
            }
        }
    }
};
</script>

<style scoped>
div {
  padding: 25%;
  background-color: #46670a;
}
.active {
  background-color: #513b20;
}
</style>
