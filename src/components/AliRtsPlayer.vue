<script setup>
import { ref, onMounted, onUnmounted, watch, onDeactivated } from 'vue';

const props = defineProps({
    src: {
        type: String,
        required: true
    },
    fallbackUrl: {
        type: String,
        default: ''
    },
    playing: {
        type: Boolean,
        default: false
    }
});

const videoRef = ref(null);
let aliRts = null;
let remoteStream = null;


// 初始化阿里云RTS播放器
const initRtsPlayer = async () => {
    if (!videoRef.value || !props.src) return;

    // 初始化SDK
    aliRts = AliRTS.createClient();

    // 监听错误事件
    aliRts.on('onError', (error) => {
        console.log(error.errorCode, error.message);
        // 降级判断
        if (error.errorCode === 10410) {
            fallback();
        }
    });

    // 检测浏览器是否支持
    try {
        const result = await aliRts.isSupport({ isReceiveVideo: true });
        // 只有在playing为true时才订阅并播放
        if (props.playing) {
            await subscribeRts();
        }
    } catch (err) {
        console.log('support error', err);
        if (props.playing) {
            fallback();
        }
    }
};

// 订阅RTS流
const subscribeRts = async () => {
    try {
        remoteStream = await aliRts.subscribe(props.src, {
            mediaTimeout: 6000,
            retryTimes: 5,
            retryInterval: 2000,
        });
        remoteStream.play(videoRef.value);
    } catch (err) {
        console.error('RTS播放失败:', err);
    }
};

// 降级至其他协议播放
const fallback = () => {
    console.log('================[降级]================');
    if (props.fallbackUrl && window.Hls && window.Hls.isSupported()) {
        if (videoRef.value.srcObject) {
            videoRef.value.srcObject = null;
        }
        const hls = new window.Hls();
        hls.attachMedia(videoRef.value);
        hls.loadSource(props.fallbackUrl);
    }
}

// 清理资源
const cleanup = () => {
    if (aliRts) {
        aliRts.unsubscribe();
    }
};

// 监听playing状态变化
watch(() => props.playing, async (isPlaying) => {
    if (isPlaying) {
        if (!aliRts) {
            await initRtsPlayer();
        } else if (remoteStream) {
            remoteStream.play(videoRef.value);
        } else {
            await subscribeRts();
        }
    } else if (remoteStream && aliRts) {
        // 暂停播放但不销毁
        aliRts.unsubscribe();
        remoteStream = null;
    }
});

onMounted(() => {
    if (props.src && props.playing) {
        initRtsPlayer();
    }
});

onUnmounted(() => {
    cleanup();
});

onDeactivated(() => {
    cleanup();
});
</script>

<template>
    <div class="ali-rts-player">
        <video ref="videoRef" muted controls autoplay></video>
    </div>
</template>

<style scoped>
.ali-rts-player {
    width: 100%;
    height: 100%;
}

video {
    width: 100%;
    height: 100%;
    background-color: #000;
}
</style>