<template>
  <div class="appBackground bg-light-blue-12 flex items-center justify-center">
    <div class="chooseSong text-center" v-if="isChooseSongShow">
      <h5>موزیک های خود را انتخاب کنید</h5>
      <div>
        <input
          ref="fileInput"
          type="file"
          accept="audio/*"
          style="display: none"
          multiple
          @input="handleFile"
        />
        <q-btn class="chooseSongBtn" @click="$refs.fileInput.click()" flat>
          <q-icon
            name="upload"
            class="chooseSongBtnIcon"
            size="32px"
            color="light-blue-12"
          />
        </q-btn>
      </div>
    </div>
    <div class="container flex items-center flex-col" v-if="!isChooseSongShow">
      <span class="musicTitle">{{ songTitle }}</span>
      <img
        class="nusicCover"
        :src="coverImage"
        v-if="coverImage"
        alt="کاور آهنگ"
      />
      <div v-else class="text-white">کاوری وجود ندارد</div>

      <div class="controls text-center mt-4">
        <q-btn dense icon="skip_previous" @click="prevSong" />
        <q-btn
          dense
          :icon="isPlaying ? 'pause' : 'play_arrow'"
          @click="togglePlay"
        />
        <q-btn dense icon="skip_next" @click="nextSong" />
      </div>

      <div
        class="progress-bar-wrapper"
        @click="seek"
        @mousedown="startSeek"
        @mouseup="stopSeek"
        @mouseleave="stopSeek"
        @mousemove="onSeekMove"
      >
        <div class="progress-bar" :style="{ width: progress + '%' }"></div>
      </div>
    </div>
  </div>
</template>

<script>
import jsmediatags from "jsmediatags/dist/jsmediatags.min.js";

export default {
  data() {
    return {
      isChooseSongShow: true,
      songs: [],
      currentIndex: 0,
      audio: null,
      isPlaying: false,
      progress: 0,
      duration: 0,
      songTitle: "",
      coverImage: null,
      isSeeking: false,
    };
  },
  methods: {
    handleFile(event) {
      const files = Array.from(event.target.files);
      if (!files.length) return;

      this.songs = files;
      this.currentIndex = 0;
      this.isChooseSongShow = false;
      this.initAudio();
    },

    initAudio() {
      if (this.audio) {
        this.audio.pause();
        this.audio = null;
      }

      const currentFile = this.songs[this.currentIndex];
      this.audio = new Audio(URL.createObjectURL(currentFile));

      this.extractMeta(currentFile);

      this.audio.addEventListener("loadedmetadata", () => {
        this.duration = this.audio.duration;
      });

      this.audio.addEventListener("timeupdate", () => {
        if (!this.isSeeking) {
          this.progress = (this.audio.currentTime / this.audio.duration) * 100;
        }
      });

      this.audio.addEventListener("ended", () => {
        this.nextSong();
      });

      this.playAudio();
    },

    playAudio() {
      this.audio.play();
      this.isPlaying = true;
    },
    pauseAudio() {
      this.audio.pause();
      this.isPlaying = false;
    },
    togglePlay() {
      this.isPlaying ? this.pauseAudio() : this.playAudio();
    },

    nextSong() {
      if (this.currentIndex < this.songs.length - 1) {
        this.currentIndex++;
        this.initAudio();
      }
    },
    prevSong() {
      if (!this.audio) return;

      if (this.audio.currentTime > 3) {
        this.audio.currentTime = 0;
        this.progress = 0;
      } else if (this.currentIndex > 0) {
        this.currentIndex--;
        this.initAudio();
      } else {
        this.audio.currentTime = 0;
        this.progress = 0;
      }
    },

    seek(event) {
      if (!this.audio) return;
      const rect = event.currentTarget.getBoundingClientRect();
      const offsetX = event.clientX - rect.left;
      const percent = offsetX / rect.width;
      this.audio.currentTime = percent * this.audio.duration;
      this.progress = percent * 100;
    },

    startSeek() {
      this.isSeeking = true;
    },
    stopSeek() {
      this.isSeeking = false;
    },
    onSeekMove(event) {
      if (!this.isSeeking || !this.audio) return;
      const rect = event.currentTarget.getBoundingClientRect();
      const offsetX = event.clientX - rect.left;
      let percent = offsetX / rect.width;
      percent = Math.min(Math.max(percent, 0), 1);
      this.audio.currentTime = percent * this.audio.duration;
      this.progress = percent * 100;
    },

    extractMeta(file) {
      jsmediatags.read(file, {
        onSuccess: (tag) => {
          const tags = tag.tags;
          this.songTitle = tags.title || file.name;
          if (tags.picture) {
            const { data, format } = tags.picture;
            let base64String = "";
            data.forEach((byte) => {
              base64String += String.fromCharCode(byte);
            });
            this.coverImage = `data:${format};base64,${btoa(base64String)}`;
          } else {
            this.coverImage = null;
          }
        },
        onError: (error) => {
          console.log("Metadata error:", error);
          this.songTitle = file.name;
          this.coverImage = null;
        },
      });
    },
  },
};
</script>

<style>
.appBackground {
  width: 100%;
  height: 100dvh;
}
.container {
  width: 1000px;
  height: 500px;
  background: rgba(255, 255, 255, 0.144);
  backdrop-filter: blur(100px);
  border: 1px solid rgba(255, 255, 255, 0.755);
  border-radius: 16px;
  flex-direction: column;
  align-items: center;
  padding: 15px 0;
}

.chooseSong {
  width: 400px;
  background: rgba(255, 255, 255, 0.144);
  backdrop-filter: blur(100px);
  border: 1px solid rgba(255, 255, 255, 0.755);
  border-radius: 16px;
}
.chooseSongBtn {
  width: 50px;
  height: 50px;
  border: none;
  outline: none;
  cursor: pointer;
  background: rgb(255, 255, 255);
  backdrop-filter: blur(100px);
  border-radius: 100px;
  margin-bottom: 50px;
}
.progress-bar-wrapper {
  height: 6px;
  background: #ccc;
  width: 80%;
  cursor: pointer;
  border-radius: 3px;
  margin-top: 25px;
  user-select: none;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: #000000;
  transition: width 0.2s;
  border-radius: 3px;
}
img.nusicCover {
  border-radius: 18px;
  width: 300px;
  height: auto;
  margin-top: 10px;
  margin-bottom: 15px;
}
.musicTitle {
  font-size: 28px;
  margin-top: 25px;
  color: white;
}
.controls q-btn {
  margin: 0 10px;
  color: white;
}
.text-white {
  color: white;
}
</style>
