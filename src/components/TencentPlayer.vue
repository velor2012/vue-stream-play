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
let Tcplyaer = null;

// 初始化腾讯云Tcplayer播放器
const initRtsPlayer = async () => {

    if (!props.src) return;

    if (!Tcplyaer) {

        // 初始化SDK
        Tcplyaer = window?.TCPlayer('tcplayer-container', {
            sources: []
        });
        if (!Tcplyaer) {
            console.warn("TCPlayer 初始化失败");
            return;
        };
        console.log("TCPlayer 初始化成功", Tcplyaer);

    }

    if (props.playing) {
        Tcplyaer.src(props.src);
        debugger
    }
};

// 清理资源
const cleanup = () => {
    if (Tcplyaer) {
        Tcplyaer.unsubscribe();
    }
};

// 监听playing状态变化
watch(() => props.playing, async (isPlaying) => {
    if (isPlaying) {
        if (!Tcplyaer) {
            await initRtsPlayer();
        } else {
            Tcplyaer.src(props.src);
        }
    } else {
        // 暂停播放但不销毁
        var x = document.querySelector("#tcplayer-container video");
        x?.pause();
    }
});

onMounted(() => {
    if (props.src && props.playing) {
        initRtsPlayer();
    }
});

onUnmounted(() => {
    Tcplyaer?.dispose?.();
});

onDeactivated(() => {
    var x = document.getElementById("tcplayer-container");
    x?.pause();
});
</script>

<template>
    <div class="tencent-tcplayer relative">
        <video id="tcplayer-container" ref="videoRef" muted controls autoplay></video>
    </div>
</template>

<style scoped>
.tencent-tcplayer {
    width: 100%;
    height: 100%;
}

video {
    width: 100%;
    height: 100%;
    background-color: #000;
}
</style>
<style>
.tcplayer {
    width: 100%;
}
</style>