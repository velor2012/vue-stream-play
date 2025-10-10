<script setup>
import { ref, onMounted, onUnmounted, watch, computed, h } from 'vue';
import JessibucaPlayer from './JessibucaPlayer.vue';
import TencentPlayer from './TencentPlayer.vue';
import AliRtsPlayer from './AliRtsPlayer.vue';
import FMP4Player from './FMP4Player.vue';

const props = defineProps({
    src: {
        type: String,
        default: ''
    },
    type: {
        type: String,
        default: 'normal', // normal, aliyun, tencent, fmp4
        validator: (value) => ['normal', 'aliyun', 'tencent', 'fmp4'].includes(value)
    },
    playing: {
        type: Boolean,
        default: false
    }
});

// 根据类型选择不同的播放器组件
const currentComponent = computed(() => {
    switch (props.type) {
        case 'normal':
            return JessibucaPlayer;
        case 'aliyun':
            return AliRtsPlayer;
        case 'tencent':
            return TencentPlayer;
        case 'fmp4':
            return FMP4Player;
        default:
            return JessibucaPlayer;
    }
});
</script>

<template>
    <div class="video-player">
        <component :is="currentComponent" :src="src" :playing="playing" />
    </div>
</template>

<style scoped>
.video-player {
    width: 100%;
    height: 100%;
    min-height: 400px;
    background-color: #000;
}
</style>