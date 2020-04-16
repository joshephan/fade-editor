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
    <div class="btns">
      <fa class="icon" icon="adjust" @click="isDark = !isDark" />
      <fa class="icon" icon="file-download" @click="generate" />
      <a href="https://brunch.co.kr/@skykamja24">
        <fa class="icon" icon="user-circle" />
      </a>
    </div>
    <textarea
      :style="{ opacity: opacity, filter: `blur(${blur}px)` }"
      v-model="text"
      placeholder="Don't stop writing until finish."
    ></textarea>
  </main>
</template>
<script>
import { Document, Packer, Paragraph } from "docx";
import { saveAs } from "file-saver";

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
        this.isStart = false;
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
    },
    generate() {
      const doc = new Document();
      const paragraph = new Paragraph({ text: this.text });
      doc.addSection({
        children: [paragraph]
      });
      Packer.toBlob(doc).then(blob => {
        saveAs(blob, "fade-text.docx");
      });
    }
  }
};
</script>
<style lang="scss">
$dark: #333333;
main {
  height: 100vh;
  overflow: hidden;
  .icon {
    color: $dark;
  }
  &.dark {
    background-color: $dark;
    textarea {
      color: #eeeeee;
      background-color: $dark;
      &::-webkit-scrollbar-track {
        -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
        border-radius: 10px;
        background-color: $dark;
      }
      &::-webkit-scrollbar {
        width: 12px;
        background-color: $dark;
      }
      &::-webkit-scrollbar-thumb {
        border-radius: 10px;
        -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
        background-color: rgb(99, 99, 99);
      }
    }
    .icon {
      color: #eeeeee;
    }
  }
}
.top {
  position: sticky;
  display: flex;
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
  width: calc(100% - 30px);
  max-width: 900px;
  margin: 15px;
  padding: 10px 15px;
  box-sizing: border-box;
  box-shadow: none;
  height: 90vh;
  border: none;
  resize: none;
  outline: none;
  font-size: 20px;
  font-family: Noto Sans KR;
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
.btns {
  position: fixed;
  cursor: pointer;
  bottom: 12px;
  right: 20px;
  z-index: 1;
  .icon {
    font-size: 20px;
    float: right;
    margin: 10px 15px;
  }
}
</style>
