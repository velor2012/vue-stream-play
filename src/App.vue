<script setup lang="ts">
import { computed, ref, onMounted, watch } from 'vue';
// @ts-ignore
import VideoPlayer from './components/VideoPlayer.vue';

// 定义流类型
interface StreamType {
    id: string;
    name: string;
    type: 'normal' | 'aliyun' | 'tencent' | 'fmp4';
}

// 定义标签项（移除了url属性，因为现在共享一个URL）
interface TabItem extends StreamType { }

// 初始化标签列表
const tabs: TabItem[] = [
    { id: 'normal', name: '常规', type: 'normal' },
    { id: 'aliyun', name: '阿里云', type: 'aliyun' },
    { id: 'tencent', name: '腾讯云', type: 'tencent' },
    { id: 'fmp4', name: 'FMP4', type: 'fmp4' },
];

// 共享的URL
const sharedUrl = ref<string>('');

// 播放状态控制
const isPlaying = ref<boolean>(false);

// 当前激活的标签
const activeTab = ref<string>(tabs[0]!.id);

// 从URL参数中获取拉流地址
const getUrlParam = (name: string): string | null => {
    const urlParams = new URLSearchParams(window.location.search);
    return urlParams.get(name);
};

// 更新URL参数，不刷新页面
const updateUrlParam = (name: string, value: string) => {
    const url = new URL(window.location.href);
    url.searchParams.set(name, value);
    window.history.pushState({}, '', url.toString());
};

// 保存到localStorage
const saveToLocalStorage = () => {
    localStorage.setItem('streamPlayerUrl', sharedUrl.value);
    localStorage.setItem('streamPlayerTab', activeTab.value);
};

// 从localStorage加载
const loadFromLocalStorage = () => {
    const savedUrl = localStorage.getItem('streamPlayerUrl');
    const savedTab = localStorage.getItem('streamPlayerTab');

    if (savedUrl) {
        sharedUrl.value = savedUrl;
    }

    if (savedTab && tabs.some(tab => tab.id === savedTab)) {
        activeTab.value = savedTab;
    }
};

// 切换标签
const switchTab = (tabId: string) => {
    activeTab.value = tabId;
    // 切换标签时停止播放
    isPlaying.value = false;
    // 保存到localStorage
    saveToLocalStorage();
    // 更新URL参数
    if (sharedUrl.value) {
        updateUrlParam('tab', tabId);
    }
};

// 获取当前标签
const currentTab = computed(() => {
    return tabs.find(tab => tab.id === activeTab.value) || tabs[0]!;
});

// 开始播放
const togglePlaying = () => {
    isPlaying.value = !isPlaying.value;
};

// 新增：移动端提示状态与工具方法
const showMobileWarning = ref<boolean>(false);
let mobileWarningTimer: number | null = null;

const isMobile = (): boolean => {
    const ua = navigator.userAgent || navigator.vendor || (window as any).opera;
    return /android|iphone|ipad|ipod|mobile/i.test(ua);
};

const triggerMobileWarning = () => {
    if (mobileWarningTimer) {
        clearTimeout(mobileWarningTimer);
        mobileWarningTimer = null;
    }
    showMobileWarning.value = true;
    mobileWarningTimer = window.setTimeout(() => {
        showMobileWarning.value = false;
        mobileWarningTimer = null;
    }, 5000);
};

// 获取当前标签的提示信息
const getTabHint = (type: string) => {
    switch (type) {
        case 'normal':
            return 'MP4, WebM等 (使用Jessibuca播放)';
        case 'aliyun':
            return '阿里云RTS流 (支持降级到HLS)';
        case 'tencent':
            return '腾讯云直播流 (使用Jessibuca播放)';
        case 'fmp4':
            // 新增：进入 fmp4 分支时，若为移动端则触发提示卡片
            if (isMobile()) {
                triggerMobileWarning();
            }
            return 'FMP4流 (使用wsPlayer通过WebSocket播放)';
        default:
            return '';
    }
};

// 监听sharedUrl变化，保存到localStorage并更新URL
watch(sharedUrl, (newUrl) => {
    saveToLocalStorage();
    if (newUrl) {
        updateUrlParam('url', newUrl);
    }
});

// 页面加载时，优先从URL读取，其次从localStorage读取
onMounted(() => {
    // 从URL读取
    const urlParam = getUrlParam('url');
    const tabParam = getUrlParam('tab');

    let urlLoaded = false;

    if (urlParam) {
        sharedUrl.value = urlParam;
        urlLoaded = true;
    }

    if (tabParam && tabs.some(tab => tab.id === tabParam)) {
        activeTab.value = tabParam;
        urlLoaded = true;
    }

    // 如果URL没有参数，则从localStorage读取
    if (!urlLoaded) {
        loadFromLocalStorage();
    }
});
</script>

<template>
    <div class="app-container">
        <header class="app-header">
            <h1 class="app-title">视频流播放器</h1>
            <p class="app-subtitle">支持多种视频流格式的在线播放</p>
        </header>

        <main class="app-main">
            <!-- 标签导航 -->
            <div class="tab-nav">
                <button v-for="tab in tabs" :key="tab.id" @click="switchTab(tab.id)" class="tab-button"
                    :class="{ 'tab-active': activeTab === tab.id }">
                    {{ tab.name }}
                </button>
            </div>

            <!-- URL输入区域 -->
            <div class="url-input-container">
                <div class="input-group">
                    <input type="text" v-model="sharedUrl" placeholder="请输入视频流地址..." class="url-input" />
                    <button @click="togglePlaying" :class="['play-button', isPlaying ? 'pause-button' : '']">
                        {{ isPlaying ? '暂停' : '播放' }}
                    </button>
                </div>
                <p class="format-hint">
                    支持的格式: {{ getTabHint(currentTab.type) }}
                </p>
            </div>

            <!-- 视频播放区域 -->
            <div class="video-container">
                <keep-alive>
                    <VideoPlayer :key="currentTab.type" :src="sharedUrl" :type="currentTab.type" :playing="isPlaying" />
                </keep-alive>
            </div>

            <!-- 新增：移动端提示卡片覆盖层 -->
            <div v-if="showMobileWarning" class="mobile-warning-overlay">
                <div class="mobile-warning-card">
                    请切换到常规标签页并将wss改为http并更换后缀.mp4为.flv
                </div>
            </div>
        </main>
    </div>
</template>

<style>
@import './assets/main.css';

body {
    font-family: 'Inter', sans-serif;
    background-color: #f0f4f8;
    margin: 0;
    padding: 0;
}

.app-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
    height: 100vh;
    display: flex;
    flex-direction: column;
    animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.app-header {
    margin-bottom: 2rem;
    text-align: center;
    padding: 1rem;
    border-radius: 16px;
    background: linear-gradient(145deg, #ffffff, #f0f0f0);
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.05);
    transform: translateZ(0);
    transition: all 0.3s ease;
}

.app-header:hover {
    transform: translateY(-5px);
    box-shadow: 0 12px 28px rgba(0, 0, 0, 0.1);
}

.app-title {
    font-size: 2.5rem;
    font-weight: 700;
    color: #3b82f6;
    margin-bottom: 0.5rem;
    background: linear-gradient(to right, #3b82f6, #2563eb);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    text-shadow: 0 2px 10px rgba(59, 130, 246, 0.2);
}

.app-subtitle {
    color: #64748b;
    font-size: 1.1rem;
}

.app-main {
    flex: 1;
    display: flex;
    flex-direction: column;
    border-radius: 20px;
    background: #ffffff;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
    padding: 2rem;
}

.tab-nav {
    display: flex;
    border-bottom: 1px solid #e2e8f0;
    margin-bottom: 1.5rem;
    padding-bottom: 0.5rem;
    overflow-x: auto;
    scrollbar-width: none;
}

.tab-nav::-webkit-scrollbar {
    display: none;
}

.tab-button {
    padding: 0.75rem 1.5rem;
    font-weight: 500;
    color: #64748b;
    border: none;
    background: transparent;
    border-radius: 12px;
    margin-right: 0.5rem;
    cursor: pointer;
    transition: all 0.3s ease;
    position: relative;
}

.tab-button:hover {
    color: #3b82f6;
    background-color: rgba(59, 130, 246, 0.05);
    transform: translateY(-2px);
}

.tab-active {
    color: #3b82f6;
    font-weight: 600;
}

.tab-active::after {
    content: '';
    position: absolute;
    bottom: -0.5rem;
    left: 50%;
    transform: translateX(-50%);
    width: 30px;
    height: 3px;
    background: linear-gradient(to right, #3b82f6, #2563eb);
    border-radius: 3px;
    animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
    from {
        width: 0;
        opacity: 0;
    }

    to {
        width: 30px;
        opacity: 1;
    }
}

.url-input-container {
    margin-bottom: 1.5rem;
}

.input-group {
    display: flex;
    align-items: center;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
    flex-wrap: nowrap;
}

.input-group:focus-within {
    box-shadow: 0 6px 16px rgba(59, 130, 246, 0.15);
    transform: translateY(-2px);
}

.url-input {
    flex: 1 1 auto;
    min-width: 0;
    width: 100%;
    padding: 0.875rem 1.25rem;
    border: 1px solid #e2e8f0;
    border-right: none;
    border-radius: 12px 0 0 12px;
    font-size: 1rem;
    outline: none;
    transition: all 0.3s ease;
}

.url-input:focus {
    border-color: #3b82f6;
}

.play-button {
    padding: 0.875rem 1.5rem;
    background: linear-gradient(to right, #3b82f6, #2563eb);
    color: white;
    font-weight: 500;
    border: none;
    border-radius: 0 12px 12px 0;
    cursor: pointer;
    transition: all 0.3s ease;
    white-space: nowrap;
    flex-shrink: 0;
}

.play-button:hover {
    background: linear-gradient(to right, #2563eb, #1d4ed8);
    transform: translateX(2px);
}

.pause-button {
    background: linear-gradient(to right, #ef4444, #dc2626);
}

.pause-button:hover {
    background: linear-gradient(to right, #dc2626, #b91c1c);
}

.format-hint {
    font-size: 0.875rem;
    color: #64748b;
    margin-top: 0.5rem;
    padding-left: 0.5rem;
}

.video-container {
    /* flex: 1; */
    background-color: #0f172a;
    border-radius: 16px;
    /* overflow: hidden; */
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
    transition: all 0.3s ease;

    padding: 10px;
}

.video-container:hover {
    box-shadow: 0 12px 32px rgba(0, 0, 0, 0.18);
    transform: scale(1.005);
}

@media (max-width: 480px) {
  .url-input {
    padding: 0.75rem 1rem;
    font-size: 0.95rem;
  }
  .play-button {
    padding: 0.75rem 1rem;
    font-size: 0.95rem;
  }
}

/* 新增：移动端提示卡片样式 */
.mobile-warning-overlay {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: start;
    justify-content: center;
    z-index: 1000;
    pointer-events: none;
}

.mobile-warning-card {
    margin: 10px;
    background: #ffffff;
    color: #0f172a;
    padding: 1rem 1.25rem;
    border-radius: 12px;
    box-shadow: 0 12px 32px rgba(0, 0, 0, 0.2);
    max-width: 520px;
    text-align: center;
    font-size: 0.95rem;
    animation: fadeInUp 0.25s ease-out;
}

@keyframes fadeInUp {
    from {
        opacity: 0;
        transform: translateY(8px);
    }

    to {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>
