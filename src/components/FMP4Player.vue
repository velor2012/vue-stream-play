<script setup>
import { ref, onMounted, onUnmounted, watch, onDeactivated } from 'vue';

const props = defineProps({
    src: {
        type: String,
        required: true
    },
    playing: {
        type: Boolean,
        default: false
    }
});
let player = null;

const initPlayer = () => {
    if (!props.src) return;

    // 销毁之前的实例
    destroyPlayer();

    // 创建新的wsPlayer实例
    player = new window.wsPlayer('wsplayer', props.src);

    // 只有在playing为true时才开始播放
    if (props.playing) {
        player.open();
    }
};

const destroyPlayer = () => {
    if (player) {
        player.close();
        player = null;
    }
};

// 监听playing状态变化
watch(() => props.playing, (isPlaying) => {
    if (isPlaying) {
        if (!player) {
            initPlayer();
        } else {
            player.open();
        }
    } else if (player) {
        var x = document.getElementById("wsplayer");
        x?.pause();
    }
});

onMounted(() => {
    if (props.src && props.playing) {
        initPlayer();
    }
});

onUnmounted(() => {
    destroyPlayer();
});

onDeactivated(() => {
    destroyPlayer();
});
</script>

<template>
    <div class="fmp4-player">
        <video id="wsplayer" ref="videoRef" controls autoplay></video>
    </div>
</template>

<style scoped>
.fmp4-player {
    width: 100%;
    height: 100%;
}

video {
    width: 100%;
    height: 100%;
    background-color: #000;
}
</style>