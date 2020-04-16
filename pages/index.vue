<template>
  <main :class="[isDark ? 'dark' : '']">
    <div class="top">
      <div
        class="bar"
        :style="{
          'background-color': color,
          width: `${time >= 60 ? 100 : (time / 60) * 100}%`
        }"
      ></div>
    </div>
    <div class="btns">
      <fa class="icon" icon="adjust" @click="isDark = !isDark" />
      <fa class="icon" icon="file-download" @click="generate" />
      <a href="https://brunch.co.kr/@skykamja24/392">
        <fa class="icon" icon="user-circle" />
      </a>
      <fa class="icon" icon="question" @click="isQuestionOpen = !isQuestionOpen" />
    </div>
    <div
      v-if="isQuestionOpen"
      class="question"
    >It is an editor that can be written without stopping. Stop typing and after 60 seconds, everything disappears. If you want to save, press the Save button at the bottom. Be careful because the deleted text cannot be reversed.</div>
    <textarea
      :style="{ opacity: opacity, filter: `blur(${blur}px)` }"
      v-model="text"
      placeholder="Don't stop writing until finish."
      @keyup="type"
    ></textarea>
  </main>
</template>
<script>
import { Document, Packer, Paragraph } from "docx";
import { saveAs } from "file-saver";

export default {
  data() {
    return {
      time: 60,
      color: "#15ad5c",
      text: "",
      opacity: 1,
      blur: 0,
      isStart: false,
      isDark: false,
      isQuestionOpen: false,
      timer: null
    };
  },
  methods: {
    countDown() {
      if (!this.isStart) {
        return false;
      }
      if (this.time > 0) {
        this.time -= 0.03;
      } else {
        this.time = 0;
        this.text = "";
        this.isStart = false;
      }
      this.blur = 2 - this.time / 30;
      this.opacity = this.time / 60;
    },
    colorChange() {
      if (this.time >= 50) {
        this.color = "#15ad5c";
      } else if (this.time >= 40) {
        this.color = "#45d610";
      } else if (this.time >= 30) {
        this.color = "#d6d510";
      } else if (this.time >= 20) {
        this.color = "#d68310";
      } else {
        this.color = "#d61010";
      }
    },
    type() {
      if (!this.isStart) {
        this.isStart = true;
        this.timer = setInterval(() => {
          this.countDown();
          this.colorChange();
        }, 30);
      } else if (this.text.length === 0) {
        clearInterval(this.timer);
        this.isStart = false;
      }
      this.time = 60;
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
$dark: #171719;
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
    .question {
      background-color: #353546;
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
.question {
  position: fixed;
  bottom: 58px;
  background-color: $dark;
  border-radius: 10px;
  padding: 15px;
  z-index: 1;
  word-break: keep-all;
  right: 10px;
  width: calc(100% - 20px);
  max-width: 300px;
  color: white;
}
textarea {
  width: calc(100% - 30px);
  max-width: 900px;
  margin: 15px;
  padding: 10px 15px;
  box-sizing: border-box;
  box-shadow: none;
  height: 82vh;
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
