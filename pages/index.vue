<template>
  <main :class="[isDark ? 'dark' : '']">
    <div class="top">
      <div
        class="bar"
        :style="{
          'background-color': color,
          width: `${time >= 30 ? 100 : (time / 30) * 100}%`
        }"
      ></div>
    </div>
    <a class="night-mode" href="#" @click="isDark = !isDark">
      <fa icon="adjust" :style="{color: isDark ? 'white' : '#2222222'}"/>
    </a>
    <textarea
      :style="{ opacity: opacity, filter: `blur(${blur}px)` }"
      v-model="text"
      placeholder="Don't stop writing until finish."
    ></textarea>
  </main>
</template>
<script>
export default {
  data() {
    return {
      time: 30,
      color: "#26FF8B",
      text: "",
      opacity: 1,
      blur: 0,
      isStart: false,
      isDark: false
    };
  },
  watch: {
    text(to, from) {
      if (to.length !== from.length) {
        this.type();
      }
    }
  },
  methods: {
    countDown() {
      if (this.time > 0) {
        this.time -= 0.03;
      } else {
        this.time = 0;
        this.text = "";
      }
      this.blur = 2 - this.time / 15;
      this.opacity = this.time / 30;
    },
    colorChange() {
      if (this.time >= 25) {
        this.color = "#26FF8B";
      } else if (this.time >= 15) {
        this.color = "#FFD954";
      } else {
        this.color = "#FF4F5A";
      }
    },
    type() {
      if (!this.isStart) {
        this.isStart = true;
        setInterval(() => {
          this.countDown();
          this.colorChange();
        }, 30);
      }
      this.time = 30;
    }
  }
};
</script>
<style lang="scss">
main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  &.dark {
    background-color: #222222;
    textarea {
      color: #eeeeee;
    }
  }
}
.top {
  position: sticky;
  display: flex;
  justify-content: center;
  align-items: center;
  top: 0;
  width: 100%;
  height: 4px;
  .bar {
    width: 100%;
    height: inherit;
  }
  .time {
    position: absolute;
    font-size: 12px;
  }
}
textarea {
  width: 100%;
  max-width: 600px;
  margin: 15px;
  padding: 10px 15px;
  box-sizing: border-box;
  box-shadow: none;
  height: 90vh;
  border: none;
  resize: none;
  outline: none;
  font-size: 20px;
  &::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    border-radius: 10px;
    background-color: #f5f5f5;
  }
  &::-webkit-scrollbar {
    width: 12px;
    background-color: #f5f5f5;
  }

  &::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    background-color: #555;
  }
}
.night-mode {
  position: fixed;
  top: 16px;
  right: 16px;
  z-index: 1;
}
</style>
