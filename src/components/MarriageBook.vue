<template>
  <div class="wedding-invitation" :class="{ 'invitation-bounce':canOpen }">
    <div class="invitation-container" :class="{ 'invitation-down':isOpening }">
      <div class="invitation-cover">
        <div class="cover-inside-left" :class="{'opening':isOpening}">
          <div class="inside-text">
           <pre >
                <code v-html="highlightedCode"></code>
          </pre>
          </div>
        </div>
        <div class="cover-inside-right" :class="{'opening':isOpening}"></div>
        <img class="cover-inside-seal" src="../images/seal.png" @click="openInvitation"
             :class="{'invitation-flight':isOpening}">
      </div>
    </div>
  </div>
</template>

<script>
import Prism from "prismjs";
import data from "@/mock/data";

export default {
  props: ['canOpen'],
  data() {
    return {
      isOpening: false,
      isCursorVisible: 1,
      code: data.code,
      currentCode: '',
      canExecute: false,
    }
  },
  created() {
    this.progressivelyTyping()
  },
  computed: {
    highlightedCode () {
      const code = Prism.highlight(
          this.currentCode,
          Prism.languages.javascript
      )
      const codeWithCursor = `${code}<span style="opacity:${this.isCursorVisible}"> ▍</span>`
      return codeWithCursor
    }
  },
  methods: {
    // 代码输入
    progressivelyTyping() {
      return new Promise((resolve) => {
        let count = 0, typingCount = 0, typing
        // 写代码每一帧的函数
        let step = () => {
          let randomNumber = Math.round(Math.random() * 6)
          // 模拟打字的随机速度
          if(count % 2 === 0 && randomNumber % 4 === 0){
            this.currentCode = this.code.substring(0, typingCount)
            typingCount++
          }
          // 大约每 24 帧光标闪动一次
          if(count % 24 === 0){
            this.isCursorVisible = this.isCursorVisible === 0 ? 1 : 0
          }
          count++
          if (typingCount <= this.code.length) {
            typing = requestAnimationFrame(step)
          } else {
            resolve()
            this.canExecute = true
            cancelAnimationFrame(typing)
            this.typingAudio.pause()
            this.audioAutoPlay("bgm_audio")
          }
        }
        typing = requestAnimationFrame(step)
      })
    },
    // 打开邀请函
    openInvitation() {
      this.isOpening = true
      setTimeout(() => {
        this.$emit('onOpen')
        //this.isOpening = false
        //播放音乐
      }, 660)
    },
    closeInvitation() {
      this.isOpening = false
      setTimeout(() => {
        this.$emit('onClose')
      }, 660)
    },

  }
}
</script>
<style scoped>
.inside-text {
  /*这一段控制请帖的文字*/
  /*font-family: ;*/
  font-size: 2em; /* 字体更大 */
  font-weight: bold; /* 加粗字体 */

  text-align: center; /* 文字垂直居中 */
  /* 下面是可选的，根据设计需求添加 */
  display: flex;
  justify-content: center;
  align-items: flex-start; /* 保证文本顶部对齐 */
  transform: translateX(55px) translateY(80px); /* 第一个值控制水平偏移，第二个值控制垂直偏移，根据需要调整 */
}
</style>
<style lang="less">
.wedding-invitation {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  padding: 30px 20px;
  padding-top: 60px;
  z-index: 4;
  transform: scale(0.05);
  -webkit-transform: scale(0.05);
  opacity: 0;
  transition: transform 0.8s cubic-bezier(.26, 1.84, .39, .61), opacity 0.5s linear;
  -webkit-transition: -webkit-transform 0.8s cubic-bezier(.26, 1.84, .39, .61), opacity 0.5s linear;
  background-size: 100%;
  overflow: hidden;
  &.invitation-bounce {
    opacity: 1;
    transform: scale(1);
    -webkit-transform: scale(1);
  }
  .invitation-container {
    position: relative;
    width: 100%;
    height: 100%;
    transition: transform 0.6s cubic-bezier(0.4, 0, 1, 1);
    -webkit-transition: -webkit-transform 0.6s cubic-bezier(0.4, 0, 1, 1);
    &.invitation-down {
      transform: translateY(20px);
      -webkit-transform: translateY(20px);
    }

    .hidden {
      display: none;
    }

    .invitation-cover {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background-color: #D65047;
      border-radius: 10px;
      perspective: 500px;
      box-shadow: 0 0 20px 2px rgba(0, 0, 0, 0.15);
      .cover-content {
        position: absolute;
        top: 50px;
        left: 0;
        width: 100%;
        height: calc(100% - 40px);
        padding: 10px 20px;
        transition: transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        -webkit-transition: -webkit-transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        &.invitation-up {
          transform: translateY(-60px);
          -webkit-transform: translateY(-60px);
        }
        .content-inside {
          height: 100%;
          padding: 20px;
          color: rgba(88, 128, 187, 1);
          background-color: #fff;
          text-align: center;
          overflow: auto;
        }
      }
      .cover-inside-left {
        position: absolute;
        left: 0;
        top: 0%;
        width: 70%;
        height: 100%;
        border-radius: 10px;
        background-color: #D65047;
        box-shadow: 5px 0 10px rgba(0, 0, 0, 0.2);
        z-index: 6;
        transition: transform 0.5s, opacity 0.5s;
        transform-origin: 0 50%;
        -webkit-transform-origin: 0 50%;
        opacity: 1;
      }
      .cover-inside-left.opening {
        animation: rotateLeftAndFadeOut 1s forwards;
      }
      @keyframes rotateLeftAndFadeOut  {
        0% {
          transform: rotateY(0deg);
          opacity: 1;
        }
        50% {
          transform: rotateY(-140deg);
          opacity: 1;
        }
        100% {
          transform: rotateY(-140deg);
          opacity: 0;
        }
      }

      .cover-inside-right {
        position: absolute;
        right: 0;
        top: 0;
        width: 40%;
        height: 100%;
        border-radius: 10px;
        background-color: #D65047;
        box-shadow: -5px 0 10px rgba(0, 0, 0, 0.2);
        z-index: 5;
        transition: transform 0.5s, opacity 0.5s;
        transform-origin: 100% 50%;
        -webkit-transform-origin: 100% 50%;
        opacity: 1;
      }
      .cover-inside-right.opening {
        animation: rotateRightAndFadeOut 1s forwards;
      }
      @keyframes rotateRightAndFadeOut  {
        0% {
          transform: rotate3d(0, 1, 0, 0deg);
          opacity: 1;
        }
        50% {
          transform: rotate3d(0, 1, 0, 140deg);
          opacity: 1;
        }
        100% {
          transform: rotate3d(0, 1, 0, 140deg);
          opacity: 0;
        }
      }
      .cover-inside-seal {
        position: absolute;
        left: 70%;
        bottom: 100px;
        width: 80px;
        height: 80px;
        margin-left: -40px;
        z-index: 7;
        transform-origin: 50% 50%;
        -webkit-transform-origin: 50% 50%;
        transition: all 0.8s cubic-bezier(0.4, 0, 1, 1);
        -webkit-transition: all 0.8s cubic-bezier(0.4, 0, 1, 1);
        &.invitation-flight {
          opacity: 0;
        }
      }
    }
  }
}
</style>
<style scoped>
.modal-header button {
  border: none;
  font-size: 20px;
  background: transparent;
  cursor: pointer;
  outline: none;
  margin-bottom: -4px; /* 添加这一行，根据需要调整负值的大小 */
}
.modal-body button {
  font-size: 20px;
  color: #a9895d;
  background: #f7debb;
  cursor: pointer;
  outline: none;
  border-radius: 5px; /* 增加这一行，设置较大的圆角值 */
  margin: 0.5rem; /* 添加此行以设置按钮之间的间距 */
  padding: 5px 10px; /* 可选：调整按钮内边距 */
  border: 1px solid #ccc; /* 添加这一行，设置边框细薄且颜色为灰色，可根据需要调整颜色 */
}
</style>
