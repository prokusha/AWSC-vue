<script setup>

import { reactive, ref } from 'vue'

const ws = new WebSocket('ws://localhost:9002');

const playStatus = ref(false);
const url = ref("");
const currentTime = ref(0);
const duration = ref(0);
const volume = ref(100);
const songTitle = ref("No audio");
const queueList = ref([]);

function play() {
    const audioPlayer = document.getElementById("audioPlayer");
    playStatus.value = true;
    audioPlayer.play();
}

function stop() {
    const audioPlayer = document.getElementById("audioPlayer");
    playStatus.value = false;
    audioPlayer.pause();
}

function updateSlider() {
    const audioPlayer = document.getElementById("audioPlayer");
    currentTime.value = audioPlayer.currentTime;
    duration.value = audioPlayer.duration;
}

function seekAudio() {
    const audioPlayer = document.getElementById("audioPlayer");
    audioPlayer.currentTime = currentTime.value;
}

function seekVolume() {
    const audioPlayer = document.getElementById("audioPlayer");
    audioPlayer.volume = volume.value * 0.01;
}

function formatTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = Math.floor(seconds % 60);
    return `${hours > 0 ? hours + ':' : ''}${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
}

ws.onopen = function() {
    console.log('Connected to the server');
};

ws.onmessage = function(event) {
    try {
        const data = JSON.parse(event.data);
        const audioPlayer = document.getElementById("audioPlayer");
        switch (data.type) {
            case 'play':
                audioPlayer.src = data.url;
                play();
                break;

            case 'queue':
                queueList.value = data.items;
                break;

            case 'timestamp':
                const timestamp = parseFloat(data.timestamp);
                // timestampsDiv.innerHTML += `<p>Timestamp: ${timestamp} seconds</p>`;
                audioPlayer.currentTime = timestamp;
                break;

            default:
                console.warn('Неизвестный тип сообщения:', data.type);
                break;
        }
    } catch (error) {
        console.error('Ошибка при обработке сообщения:', error);
    }
};

ws.onclose = function() {
    console.log('Disconnected from the server');
};

ws.onerror = function(error) {
    console.error('WebSocket error:', error);
};

function sendMessage(type, data = {}) {
    const message = { type: type, ...data };
    ws.send(JSON.stringify(message));
}

</script>

<template>
    <audio id="audioPlayer" 
        @timeupdate="updateSlider"
        @play="sendMessage('play')"
        @ended="sendMessage('next')">
    </audio>
    <div class="urlPlace">
        <input type="text" id="urlInput" v-model="url" placeholder="Input URL" />
        <button 
            @click="url ? 
                    sendMessage('url', { url: url }) :
                    console.warn('Empty form')">
                Send URL
        </button>
    </div>
    <div class="playerTitle">
        <span>{{ songTitle }}</span>
    </div>
    <div class="player">
        <div class="playerButtons">
            <button v-if="!playStatus" @click="play">&#9205;</button>
            <button v-else @click="stop">&#9208;</button>
            <button @click="sendMessage('sync')">&#128260;</button>
            <button @click="sendMessage('next')">&#9197;</button>
        </div>
        <div class="playerSlider">
            <span>{{ formatTime(currentTime) }}</span>
            <input type="range" @input="seekAudio" v-model="currentTime" :max="duration">
            <span>{{ formatTime(duration) }}</span>
        </div>
        <div class="playerVolume">
            <input type="range" @input="seekVolume" v-model="volume" :max="100">
            <span>{{ volume }}</span>
        </div>
    </div>
    <div class="playerList">
        <p>Queue</p>
        <p v-for="queue in queueList">{{ queue.author }} - {{ queue.title }}</p>
    </div>
</template>

<style>

.playerTitle {
    display: flex;
    justify-content: center;
    font-size: 25px;
    margin-bottom: 25px;
}

.player {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #2d3538;
}

.player span {
    color: white;
}

.player div {
    margin-left: 2rem;
    margin-right: 2rem;
}

.playerButtons {
    display: flex;
    align-items: center;
    padding-left: 0;
    padding-right: 0;
    height: 40px;
}

.playerButtons button {
    cursor: pointer;
    font-size: 20px;
    margin: 2px;
    width: 30px;
    height: 30px;
    border-radius: 50%;
    border-style: none;
    background-color: #ffffff;
}

.playerButtons button:hover {
    background-color: #afafaf;
}

.playerSlider {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 90%;
    height: 40px;
}

.playerSlider input {
    width: 90%;
    margin-left: 20px;
    margin-right: 20px;
}

.playerVolume {
    display: flex;
    align-items: center;
    height: 40px;
    width: 20%;
}

.playerVolume input {
    width: 70%;
}

.playerList {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    font-size: 25px;
}

</style>
