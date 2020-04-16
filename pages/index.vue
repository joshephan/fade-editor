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
      <fa class="icon" icon="adjust" @click="toggleNightMode" />
      <fa class="icon" icon="file-download" @click="generate" />
      <fa class="icon" icon="question" @click="isQuestionOpen = !isQuestionOpen" />
    </div>
    <transition name="fade">
      <div
        v-if="isQuestionOpen"
        class="question"
      >작성을 시작하면 60초의 시간이 조금씩 줄어듭니다. 타이핑을 하면 시간이 다시 늘어납니다. 시간이 모두 사라지면 작성한 글이 모두 사라져버리니, 꼭 저장하세요.</div>
    </transition>
    <transition name="fade">
      <textarea
        v-if="isLoad"
        :style="{ opacity: opacity, filter: `blur(${blur}px)` }"
        v-model="text"
        :placeholder="placeholder[phIdx]"
        @keyup="type"
      ></textarea>
    </transition>
  </main>
</template>
<script>
import { Document, Packer, Paragraph } from "docx";
import { saveAs } from "file-saver";

export default {
  data() {
    return {
      isLoad: false,
      time: 60,
      color: "#15ad5c",
      text: "",
      opacity: 1,
      blur: 0,
      isStart: false,
      isDark: false,
      isQuestionOpen: false,
      timer: null,
      placeholder: [
        "글을 쓰고 싶다면 정말로 뭔가를 창조하고 싶다면 넘어질 위험을 감수해야한다. - 알레그레 굿맨",
        "때로는 쓰기 싫어도 계속 써야 한다. - 스티븐 킹",
        "첫 줄을 쓰는 것은 어마어마한 공포이자 마술이며, 기도인 동시에 수줍음이다. - 존 스타인벡",
        "양이 곧 재능이다. - 나카타니 아키히로",
        "글을 쓰고 있어야 좋은 아이디어가 떠오른다. - 김우중",
        "글쓰기는 아무것도 아니다. 당신이 할 것은 타자기 앞에 앉아서 피를 흘리는 것이다. - 어니스트 헤밍웨이",
        "무슨 일이든 글쓰기부터 시작하라. 물은 수도꼭지가 켜질 때까지 흐르지 않는다. - 루이스 라모르",
        "글을 쓰기 전에는 항상 내 앞에 마주 앉은 누군가에게 이야기를 해 주는 것이라고 상상하라. 그리고 그 사람이 지루해 자리를 뜨지 않도록 설명하라. - 제임스 패터슨",
        "제대로 쓰려 말고, 무조건 써라 - 제임스 써버",
        "글을 쓸 계획을 세우지 마라. 그냥 써라. 독창적인 문체는 오로지 글을 쓸 때만 가능하다. - P. D. 제임스",
        "작가라면 그 누구든 결국 빈 공책이나 모니터 화면을 바라보아야 한다. 문장을 떠올리기 위해서라면 방망이로 자기 머리라도 내리쳐야 한다. - 오클리 홀"
      ],
      phIdx: 0
    };
  },
  created() {
    this.isDark = this.$cookies.get("dark");
    this.phIdx = Math.floor(Math.random() * this.placeholder.length);
  },
  mounted() {
    this.isLoad = true;
  },
  methods: {
    toggleNightMode() {
      this.isDark = !this.isDark;
      this.$cookies.set("dark", this.isDark, {
        path: "/",
        maxAge: 60 * 60 * 24 * 7
      });
    },
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
      } else if (this.time > 0) {
        this.color = "#d61010";
      } else {
        this.color = "gray";
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
$white: #f3f3ed;
$dark: #171719;
$icon: #48484e;
main {
  height: 100vh;
  overflow: hidden;
  background: $white;
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
      &:focus {
        color: #ac4fe6;
        text-shadow: 0px 0px 0px $white;
        -webkit-text-fill-color: auto;
      }
      &::selection {
        background-color: #ac4fe6;
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
  font-size: 18px;
  font-family: Noto Sans KR;
  background: $white;
  &::placeholder {
    color: #777;
  }
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
    text-shadow: 0px 0px 0px $dark;
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
