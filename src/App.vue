<script setup lang="ts">
import { ref } from 'vue';
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
const tabs = ref<TabItem[]>([
    { id: 'normal', name: '常规', type: 'normal' },
    { id: 'aliyun', name: '阿里云', type: 'aliyun' },
    { id: 'tencent', name: '腾讯云', type: 'tencent' },
    { id: 'fmp4', name: 'FMP4', type: 'fmp4' },
]);

// 共享的URL
const sharedUrl = ref<string>('');

// 播放状态控制
const isPlaying = ref<boolean>(false);

// 当前激活的标签
const activeTab = ref<string>(tabs.value[0].id);

// 切换标签
const switchTab = (tabId: string) => {
    activeTab.value = tabId;
    // 切换标签时停止播放
    isPlaying.value = false;
};

// 获取当前标签
const getCurrentTab = () => {
    return tabs.value.find(tab => tab.id === activeTab.value) || tabs.value[0];
};

// 开始播放
const togglePlaying = () => {
    isPlaying.value = !isPlaying.value;
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
            return 'FMP4流 (使用wsPlayer通过WebSocket播放)';
        default:
            return '';
    }
};
</script>

<template>
    <div class="container mx-auto p-4 h-screen flex flex-col overflow-y-auto">
        <header class="mb-6">
            <h1 class="text-3xl font-bold text-center text-blue-600">视频流播放器</h1>
            <p class="text-center text-gray-600">支持多种视频流格式的在线播放</p>
        </header>

        <main class="flex-1 flex flex-col">
            <!-- 标签导航 -->
            <div class="flex border-b mb-4 ">
                <button v-for="tab in tabs" :key="tab.id" @click="switchTab(tab.id)" class="px-4 py-2 font-medium"
                    :class="activeTab === tab.id ? 'text-blue-600 border-b-2 border-blue-600' : 'text-gray-500 hover:text-gray-700'">
                    {{ tab.name }}
                </button>
            </div>

            <!-- URL输入区域 -->
            <div class="mb-6 ">
                <div class="flex items-center">
                    <input type="text" v-model="sharedUrl" placeholder="请输入视频流地址..."
                        class="flex-1 p-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500" />
                    <button @click="togglePlaying"
                        :class="`${isPlaying ? 'bg-red-600' : 'bg-blue-600'} text-white px-4 py-2 rounded-r hover:bg-blue-700 focus:outline-none`">
                        {{ isPlaying ? '暂停' : '播放' }}
                    </button>
                </div>
                <p class="text-sm text-gray-500 mt-1">
                    支持的格式: {{ getTabHint(getCurrentTab().type) }}
                </p>
            </div>

            <!-- 视频播放区域 -->
            <div class="bg-black rounded-lg flex-1">
                <keep-alive>
                    <VideoPlayer :key="getCurrentTab().type" :src="sharedUrl" :type="getCurrentTab().type"
                        :playing="isPlaying" />
                </keep-alive>
            </div>
        </main>
    </div>
</template>

<style>
@import './assets/main.css';

body {
    font-family: 'Inter', sans-serif;
    background-color: #f9fafb;
}
</style>
