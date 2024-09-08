<template>
  <div id="app">
    <h1>Opus播放器</h1>
    <OpusPlayer ref="player" />
    <div>
      <h3>添加音频文件:</h3>
      <input type="text" v-model="newFileUrl" placeholder="输入opus文件URL">
      <input type="text" v-model="newFileName" placeholder="输入文件名">
      <textarea v-model="newFileText" placeholder="输入字幕文本"></textarea>
      <button @click="addFile">添加文件</button>
    </div>
  </div>
</template>

<script>
import OpusPlayer from './components/OpusPlayer.vue';

export default {
  name: 'App',
  components: {
    OpusPlayer
  },
  data() {
    return {
      newFileUrl: '',
      newFileName: '',
      newFileText: ''
    };
  },
  methods: {
    addFile() {
      if (this.newFileUrl && this.newFileName && this.newFileText) {
        this.$refs.player.addAudioFile(this.newFileUrl, this.newFileName, this.newFileText);
        this.newFileUrl = '';
        this.newFileName = '';
        this.newFileText = '';
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

textarea {
  width: 100%;
  height: 100px;
  margin-bottom: 10px;
}
</style>