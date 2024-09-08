<template>
  <div class="opus-player">
    <div class="controls">
      <button @click="togglePlay">{{ isPlaying ? '暂停' : '播放' }}</button>
      <input 
        type="range" 
        :min="0" 
        :max="totalDuration" 
        :value="currentTime" 
        @input="seek"
      >
      <span>{{ formatTime(currentTime) }} / {{ formatTime(totalDuration) }}</span>
    </div>
    <div class="subtitles">
      <p 
        v-for="(subtitle, index) in audioFiles" 
        :key="index"
        :class="{ 'active': currentPlayerIndex === index }"
        @click="jumpToSubtitle(index)"
      >
        {{ subtitle.text }}
      </p>
    </div>
    <div class="file-list">
      <h3>音频文件列表:</h3>
      <ul>
        <li v-for="(file, index) in audioFiles" :key="index">
          {{ file.name }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { Howl, Howler } from 'howler';

export default {
  name: 'OpusPlayer',
  data() {
    return {
      audioFiles: [],
      players: [],
      currentPlayerIndex: 0,
      isPlaying: false,
      currentTime: 0,
      totalDuration: 0,
      updateInterval: null,
    };
  },
  methods: {
    addAudioFile(url, name, text) {
      this.audioFiles.push({ url, name, text });
      this.initPlayer(url);
    },
    initPlayer(url) {
      const player = new Howl({
        src: [url],
        format: ['opus'],
        onload: () => {
          this.updateTotalDuration();
        },
        onend: () => {
          this.playNext();
        },
      });
      this.players.push(player);
    },
    togglePlay() {
      if (this.isPlaying) {
        this.pause();
      } else {
        this.play();
      }
    },
    play() {
      if (this.players.length === 0) return;
      // 确保只有当前音频在播放
      this.players.forEach((player, index) => {
        if (index === this.currentPlayerIndex) {
          player.play();
        } else {
          player.stop();
        }
      });
      this.isPlaying = true;
      this.startUpdateInterval();
    },
    pause() {
      if (this.players.length === 0) return;
      this.players[this.currentPlayerIndex].pause();
      this.isPlaying = false;
      this.stopUpdateInterval();
    },
    playNext() {
      this.players[this.currentPlayerIndex].stop(); // 停止当前音频
      this.currentPlayerIndex++;
      if (this.currentPlayerIndex >= this.players.length) {
        this.currentPlayerIndex = 0;
        this.isPlaying = false;
        this.stopUpdateInterval();
      } else {
        this.play();
      }
    },
    seek(event) {
      const seekTime = parseFloat(event.target.value);
      this.setCurrentTime(seekTime);
    },
    setCurrentTime(time) {
      let accumulatedDuration = 0;
      for (let i = 0; i < this.players.length; i++) {
        const duration = this.players[i].duration();
        if (accumulatedDuration + duration > time) {
          // 停止所有正在播放的音频
          this.players.forEach(player => player.stop());
          
          this.currentPlayerIndex = i;
          const seekTime = time - accumulatedDuration;
          this.players[i].seek(seekTime);
          if (this.isPlaying) {
            this.players[i].play();
          }
          break;
        }
        accumulatedDuration += duration;
      }
      this.currentTime = time;
    },
    updateTotalDuration() {
      this.totalDuration = this.players.reduce((total, player) => total + player.duration(), 0);
    },
    startUpdateInterval() {
      this.updateInterval = setInterval(() => {
        let currentTime = 0;
        for (let i = 0; i < this.currentPlayerIndex; i++) {
          currentTime += this.players[i].duration();
        }
        currentTime += this.players[this.currentPlayerIndex].seek();
        this.currentTime = currentTime;
      }, 1000);
    },
    stopUpdateInterval() {
      clearInterval(this.updateInterval);
    },
    formatTime(seconds) {
      const minutes = Math.floor(seconds / 60);
      const remainingSeconds = Math.floor(seconds % 60);
      return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`;
    },
    jumpToSubtitle(index) {
      if (index !== this.currentPlayerIndex) {
        // 停止所有正在播放的音频
        this.players.forEach(player => player.stop());
        
        // 计算新的播放位置
        let newTime = 0;
        for (let i = 0; i < index; i++) {
          newTime += this.players[i].duration();
        }
        
        this.currentPlayerIndex = index;
        this.setCurrentTime(newTime);
        
        if (this.isPlaying) {
          this.players[index].play();
        }
      }
    },
  },
  beforeUnmount() {
    this.stopUpdateInterval();
    this.players.forEach(player => player.unload());
  },
};
</script>

<style scoped>
.opus-player {
  max-width: 500px;
  margin: 0 auto;
}

.controls {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.controls button {
  margin-right: 10px;
}

.controls input[type="range"] {
  flex-grow: 1;
  margin-right: 10px;
}

.subtitles {
  margin-bottom: 20px;
}

.subtitles p {
  padding: 5px;
  margin: 5px 0;
  border-radius: 5px;
  transition: background-color 0.3s ease;
  cursor: pointer; /* 添加这行，使鼠标悬停时显示为手型 */
}

.subtitles p:hover {
  background-color: #f0f0f0; /* 添加这个样式，使鼠标悬停时有轻微的背景色变化 */
}

.subtitles p.active {
  background-color: #e0e0e0;
}

.file-list {
  margin-top: 20px;
}
</style>