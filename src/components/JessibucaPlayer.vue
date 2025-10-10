<template>
    <div class="jessibuca-player" id="container" ref="container"></div>
</template>

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

const container = ref(null);
let jessibuca = null;

const initPlayer = () => {
    if (!container.value || !props.src) return;

    // 如果已经有实例但URL变了，需要销毁重建
    if (jessibuca) {
        destroyPlayer();
    }

    // 创建新的Jessibuca实例
    jessibuca = new window.Jessibuca({
        container: container.value,
        videoBuffer: 0.2,
        isResize: true,
        text: "",
        loadingText: "加载中",
        debug: true,
        showBandwidth: false,
        operateBtns: {
            fullscreen: true,
            screenshot: true,
            play: true,
            audio: true,
        },
        forceNoOffscreen: false,
        isNotMute: false,
    });

    // 添加基本事件监听
    jessibuca.on("load", function () {
        console.log("Jessibuca loaded");
    });

    jessibuca.on("play", function () {
        console.log("Jessibuca playing");
    });

    jessibuca.on("error", function (error) {
        console.error("Jessibuca error:", error);
    });

    // 只有在playing为true时才开始播放
    if (props.playing) {
        jessibuca.play(props.src);
    }
};

// 销毁播放器实例
const destroyPlayer = () => {
    debugger
    if (jessibuca) {
        jessibuca.destroy();
        jessibuca = null;
    }
};

// 监听playing状态变化
watch(() => props.playing, (isPlaying) => {
    if (isPlaying) {
        // 如果还没有创建实例，先创建
        if (!jessibuca) {
            initPlayer();
        } else {
            // 如果已经有实例，直接播放
            jessibuca.play(props.src);
        }
    } else if (jessibuca) {
        // 停止播放但不销毁实例
        jessibuca.pause();
    }
});

onMounted(() => {
    // 只有在playing为true时才初始化
    if (props.playing && props.src) {
        initPlayer();
    }
});

onMounted(() => {
    if (props.src) {
        initPlayer();
    }
});

onUnmounted(() => {
    destroyPlayer();
});

onDeactivated(() => {
    jessibuca?.pause?.();
});

</script>

<style scoped>
.jessibuca-player {
    width: 100%;
    height: 100%;
}

div[ref="container"] {
    width: 100%;
    height: 100%;
    background-color: #000;
}
</style>
