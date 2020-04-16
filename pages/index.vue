<template>
  <main :class="[isDark ? 'dark' : '']">
    <div class="top">
      <div
        class="bar"
        :style="{
          'background-color': color,
          width: `${time >= 60 || time === 0 ? 100 : (time / 60) * 100}%`
        }"
      ></div>
    </div>
    <div class="btns">
      <fa class="icon" icon="adjust" @click="isDark = !isDark" />
      <fa class="icon" icon="file-download" @click="generate" />
      <fa class="icon" icon="question" @click="isQuestionOpen = !isQuestionOpen" />
    </div>
    <transition name="fade">
      <div
        v-if="isQuestionOpen"
        class="question"
      >작성을 시작하면 60초의 시간이 조금씩 줄어듭니다. 타이핑을 하면 시간이 다시 늘어납니다. 시간이 모두 사라지면 작성한 글이 모두 사라져버리니, 꼭 저장하세요.</div>
    </transition>
    <textarea
      :style="{ opacity: opacity, filter: `blur(${blur}px)` }"
      v-model="text"
      placeholder="끝날 때까지 멈추지 말고 작성하세요."
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
      timer: null,
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
      } else if (this.time > 0){
        this.color = "#d61010";
      } else {
        this.color = 'gray';
        this.opacity = 1;
        this.blur = 0;
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
      if (this.time < 60) {
        this.time += 1;
      }
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
$icon: #48484e;
main {
  height: 100vh;
  overflow: hidden;
  .icon {
    color: $icon;
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
  font-size: 12px;
  word-break: keep-all;
  right: 10px;
  width: calc(100% - 20px);
  max-width: 300px;
  color: white;
}
textarea {
  width: calc(100% - 30px);
  max-width: 850px;
  margin: 15px auto;
  padding: 10px 15px;
  box-sizing: border-box;
  box-shadow: none;
  height: 82vh;
  display: block;
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
  &:focus {
    color: #15ad5c;
    text-shadow: 0px 0px 0px #555555;
    -webkit-text-fill-color: transparent;
  }
  &::selection {
    background-color: #15ad5c;
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
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>
