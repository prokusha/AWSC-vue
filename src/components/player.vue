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
        const datas = JSON.parse(event.data);
        const audioPlayer = document.getElementById("audioPlayer");
        const cover = document.getElementById("cover");
        if (Array.isArray(datas)) {
            datas.forEach(data => {
                jsonParse(data);
            });
        } else {
            jsonParse(datas);
        }
    } catch (error) {
        console.error('Ошибка при обработке сообщения:', error);
    }
};

function jsonParse(data) {
    switch (data.type) {
        case 'play':
            audioPlayer.src = data.url;
            songTitle.value = data.author + " - " + data.title;
            cover.src = 'https:' + data.cover.replace("%%", "400x400");
            cover.style = 'visibility: visible;';
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
}

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

function sendURL() {
    if (url.value) {
        let surl = url.value.split("?");
        sendMessage('url', { url: surl[0] });
        url.value = "";
    }
    else console.warn('Empty input')
}

function sendNextEnded() {
    audioPlayer.src = '';
    songTitle.value = 'No audio';
    cover.src = '';
    cover.style = 'visibility: hidden;';
    sendMessage('ended');
}

</script>

<template>
    <audio id="audioPlayer" 
        @timeupdate="updateSlider"
        @play="sendMessage('play')"
        @ended="sendNextEnded">
    </audio>
    <div class="urlPlace">
        <input type="text" id="urlInput" v-model="url" placeholder="Input URL" />
        <button @click="sendURL">
            Send URL
        </button>
    </div>
    <div class="player">
        <div class="playerCover">
            <img id="cover" alt="">
        </div>
        <div class="playerTitle">
            <span>{{ songTitle }}</span>
        </div>
        <div class="playerControls">
            <div class="playerButtons">
                <button v-if="!playStatus" @click="play">&#9205;</button>
                <button v-else @click="stop">&#9208;</button>
                <button @click="sendMessage('sync')">&#128260;</button>
                <button @click="sendMessage('next')">&#9197;</button>
            </div>
            <div class="playerSlider">
                <span>{{ formatTime(currentTime) }}</span>
                <input type="range" @input="seekAudio" v-model="currentTime" :max="duration" disabled>
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
    </div>
</template>

<style>

.urlPlace {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}

.urlPlace input {
    width: 70%;
    margin: 10px;
}

.urlPlace button {
    width: 30%;
    height: 45px;
}

.playerTitle {
    display: flex;
    justify-content: center;
    font-size: 25px;
    margin-top: 25px;
    margin-bottom: 25px;
}

.playerCover {
    display: flex;
    justify-content: center;
    margin: 15px;
}

.player {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}

.player span {
    color: white;
}

.player div {
    margin-left: 2rem;
    margin-right: 2rem;
}

.playerControls {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
}

.playerButtons {
    display: flex;
    align-items: center;
    padding-left: 0;
    padding-right: 0;
    height: 50px;
}

.playerButtons button {
    cursor: pointer;
    font-size: 30px;
    margin: 10px;
    width: 60px;
    height: 60px;
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
    width: 95%;
    height: 40px;
    margin-top: 25px;
}

.playerSlider input {
    width: 100%;
    margin-left: 20px;
    margin-right: 20px;
}

.playerVolume {
    display: flex;
    align-items: center;
    flex-direction: column;
    height: 40px;
    width: 50%;
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
    color: white;
    width: 90%;
    height: 30%;
    border: 2px;
    border-radius: 25px;
    border-style: solid;
    border-color: white;
    margin-top: 25px;
}

</style>
