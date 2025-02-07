<template>
  <div id="editor" :style="background" class="wedding-editor" ref="editor">
    <audio id="typing_audio" ref="typingAudio" :src="require('../assets/typing.mp3')" loop autoplay preload ></audio>
    <audio id="bgm_audio" ref="bgmAudio" :src="require('../assets/bgm.mp3')"  loop ></audio>
    <div >
      <!--代码编辑区-->
      <pre v-show="isShow.code" >
         <code v-html="highlightedCode"></code>
      </pre>
      <div  v-show="isShow.video" :class="{'video-container':isShow.video}" ref="videoContainer">
        <video
            id="main_video"
            ref="videoPlayer"
            height="100%"
            width="100%"
            :muted="video.isMuted"
            :autoplay="video.isPlaying"
            :loop="video.isLoop"
            playsinline >
          <!--在线oss-->
          <source :src="require('../video/video.mp4')"  type="video/mp4"><!--本地-->
          您的浏览器不支持 video 标签。
        </video>
        <!-- 新增一个div包裹输入框和按钮 -->
        <div class="barrage-form">
          <button
              class="barrage-button"
              @click="handleInputWish"
          >留下您的祝福&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</button>
          <transition name="fade">
            <!-- 弹出模态框 -->
            <div v-if="isShow.transition" class="modal-mask">
              <div class="modal-dialog">
                <div class="modal-header">
                  <h3> </h3>
                  <button @click="handleCloseWish">&times;</button>
                </div>
                <div class="modal-body">
                  <div class="wish-container">
                    <h3>祝你们：</h3>
                    <input v-model="wish" placeholder="输入祝福内容">
                    <button @click="reWish"> &circlearrowright; </button>
                  </div>
                  <div class="wish-container">
                    <h3>而我是：</h3>
                    <input v-model="name" placeholder="输入你的名字">
                    <button @click="reName"> &circlearrowright; </button>
                  </div>
                  <br>
                  <div class="button-container">
                    <button @click="sendBarrage">只发个弹幕</button>
                    <button @click="sendSms">再发个短信</button>
                  </div>
                </div>
              </div>
            </div>
          </transition>

        </div>
      </div>
    </div>
    <!-- 打开邀请函按钮 -->
    <div
        class="editor-open"
        v-if="isShow.button"
        @click="openInvitation"
    >
      <span class="inner-text">
        囍
      </span>
    </div>
    <!--邀请函部分-->
    <invitation :canOpen="isShow.invitation"  @onOpen="onAfterOpenInvitation" @onClose="onAfterCloseInvitation" @sendBarrage="onAfterSending"/>
    <!--弹幕-->
    <Barrage   :wish="wish" :canStart="isShow.barrage"
             class="barrage-layer"/><!--class="barrage-layer"-->
  </div>
</template>

<script>
import Prism from 'prismjs'
import 'prismjs/themes/prism-okaidia.css'
import '../utils/raf'
import data from '../mock/data'
import Executions from './Executions'
import Invitation from './Invitation'
import Barrage from './Barrage'
// import axios from 'axios';
export default {
  //打字区
  name: 'Editor',
  // eslint-disable-next-line vue/no-unused-components
  components: { Executions, Invitation, Barrage },
  data() {
    return {
// 顶部导航背景图片配置
      background: {
        // 背景图片地址
       backgroundImage: 'url(' + require('../images/bj.jpg') + ')',
        //线上
        // 背景图片是否重复
        backgroundRepeat: 'no-repeat',
        // 背景图片大小
        backgroundSize: 'cover',
        // 背景颜色
        backgroundColor: '#000',
        // 背景图片位置
        backgroundPosition: 'center top'
      },
      isShow:{//组件是否显示
        video:false,//视频 默认不显示
        invitation:false,//邀请函 默认不显示
        button:true,//按钮 默认显示
        barrage:false,  //弹幕
        code:true,//  代码区域
        execute:true,//  代码区域
        transition:false,//  弹窗
        input:false//  输入框区域
      },
      video:{//视频配置
        isMuted:true,//是否静音 默认静音
        isPlaying:true,//是否播放 默认播放
        isLoop:true,//是否循环 默认播放
      },

      media:{
        //背景音乐
        bgm:null,
        //扣字声音
        typing:null,
        //视频
        video:null
      },
      code: data.code,
      barrages: data.barrages,
      currentCode: '',
      isCursorVisible: 1,
      wish: '',
      name: '',
    }
  },
  created() {
    this.progressivelyTyping()
    const height = document.documentElement.clientHeight;
    // 在页面整体加载完毕时
    window.onload = function () {
      // 把获取到的高度赋值给根div
      document.getElementById('editor').style.height = `${height}px`;
    };
  },
  updated() {
    this.scrollToBottom()
  },

  mounted() {
    this.playTypingAudio();//播放扣字音频
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
    handleInputWish(){
     this.reWish();
     this.reName();
      this.isShow.transition=true
      this.isShow.button= false
    },
    handleCloseWish(){
      this.isShow.transition = false
      this.isShow.button = true
    },
    reWish(){
      // 生成一个随机索引
      const randomIndex = Math.floor(Math.random() * this.barrages.length);
      // 将 this.wish 设置为随机选中的 barrages 元素
      this.wish = this.barrages[randomIndex];
    },
    reName(){
      // 生成一个随机索引
      let names = [
          '你的好闺蜜','张德美','你的好兄弟','有容','暗恋你的...','天霸','小帅','花桥第一深情','欠我20还没还','扛把子','开黑队友',
          '张三','华子','丧彪','刀子','彪子','夹板','王德法','黄黑红','大有第一帅','渠县扛把子','菽麦第一大漂亮','工商万人迷',
        '你的好姐妹','你的好基友','花中吴彦祖','小美','隔壁村王大爷'
      ];
      const randomIndex = Math.floor(Math.random() * names.length);
      this.name = names[randomIndex];
    },

    //打开邀请函面板
    openInvitation(){
      //确保音乐播放
      this.isShow.invitation = true
      this.isShow.button = false
    },
    //打开邀请函之后展开邀请后
    onAfterOpenInvitation(){
      this.playVideo()
      this.playBgmAudio();//播放背景音乐
    },
    //发送弹幕
    sendSms(){
      this.fetchData()
      this.isShow.transition = false
      window.location.href = 'sms:15520131485';
    },
    async fetchData() {
      try {
       //这里发送弹幕到自己的后台
      } catch (error) {
        this.error = error.message;
      }
    },
    //发送弹幕
    sendBarrage(){
      if (!this.wish) {
        return
      }
      this.fetchData()
      this.isShow.transition = false
      alert("感谢您的祝福！我们已经收到啦！")
      //可以调用一个后台接口 后台获取到信息和用户信息  然后写进日志
     this.onAfterSending(this.wish)
    },
    //播放视频
    playVideo(){
      this.$refs.bgmAudio.pause();//暂停音乐
      this.$refs.typingAudio.pause();//暂停音乐
      this.isShow.code =false
      this.isShow.video = true;//展示视频
      this.isShow.input = true;

      this.$refs.videoPlayer.currentTime = 0; // 将视频时间设置为起始位置
      this.video.isMuted = false;//开启声音

      this.$refs.videoPlayer.play(); // 开始播放视频
      setTimeout(() => {
        this.isShow.barrage = true;//开启弹幕
      }, 800)
    },
    //播放扣字音频
    playTypingAudio(){
      this.$refs.bgmAudio.pause();//播放音乐
      this.video.isMuted = true;//静音
      this.$refs.typingAudio.play();//播放音乐
    },
    //播放音频
    playBgmAudio(){
      this.$refs.typingAudio.pause();//暂停扣字音乐
      this.video.isMuted = true;//静音
      this.$refs.bgmAudio.play();//播放背景音乐
    },
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
            cancelAnimationFrame(typing)
            this.playBgmAudio()//播放背景音乐
            this.openInvitation()
          }
        }
        typing = requestAnimationFrame(step)
      })
    },

    audioAutoPlay(audioId) {
      let audio = document.getElementById(audioId);
      audio.play();
      document.removeEventListener("touchstart", this.audioAutoPlay);
    },
    scrollToBottom() {
      // 保持页面一直滚到最下面
      this.$refs.editor.scrollTop = 100000
    },

    //关闭邀请函后
    onAfterCloseInvitation(){
      //关闭邀请函
      this.isShow.invitation = false;
      this.isShow.button = true;//打开按钮
      this.playVideo();//播放视频
    },
    // 发送弹幕之后
    onAfterSending(wish) {
      this.wish = wish
      this.onAfterCloseInvitation()
    }
  }
}
</script>
<style scoped>
.modal-mask {
  position: fixed;
  z-index: 999;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-dialog {
  min-width: 350px;
  max-width: 400px;
  min-height: 280px;
  max-height: 350px;
  margin: 50% auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 5px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-header button {
  border: none;
  font-size: 20px;
  background: transparent;
  cursor: pointer;
  outline: none;
  margin-bottom: -4px; /* 添加这一行，根据需要调整负值的大小 */
}
.modal-body input {
  font-size: 20px;
  width: 205px;
  color: #a9895d;
  cursor: pointer;
  outline: none;
  border-radius: 5px; /* 增加这一行，设置较大的圆角值 */
  padding: 5px 10px; /* 可选：调整按钮内边距 */
  border: 1px solid #ccc; /* 添加这一行，设置边框细薄且颜色为灰色，可根据需要调整颜色 */
}
.modal-body button {
  font-size: 20px;
  color: #a9895d;
  background: #f7debb;
  cursor: pointer;
  outline: none;
  border-radius: 5px; /* 增加这一行，设置较大的圆角值 */
  margin: 1rem; /* 添加此行以设置按钮之间的间距 */
  padding: 5px 10px; /* 可选：调整按钮内边距 */
  border: 1px solid #ccc; /* 添加这一行，设置边框细薄且颜色为灰色，可根据需要调整颜色 */
}
</style>
<style lang="less">
.bgBackground{
  width:100%;
  height:100%;
}    .wish-container {
       display: flex;
       align-items: center;
       justify-content: center;
       max-width: 600px; /* 设置最大宽度，避免内容过宽 */
       margin: 0 auto; /* 居中显示 */
     }
.wish-container h3 {
  white-space: nowrap;
  margin: 0;
  font-size: 20px;
}
.wish-container input {
  flex: 1;
  padding: 10px;
  font-size: 16px;
  margin-right: 0px; /* 为输入框添加右边距 */
}
.wish-container button {
  padding: 10px 20px;
  cursor: pointer;
  font-size: 16px;
  flex-shrink: 0; /* 防止按钮缩小 */
}
.button-container {
  text-align: center; /* 水平居中 */
  margin-top: 10px; /* 上边距 */
}
.button-container button {
  margin: 0 10px; /* 按钮之间的间距 */
  padding: 10px 20px; /* 按钮的内边距 */
  cursor: pointer; /* 鼠标指针 */
}
.video-container {
  position: relative;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
.barrage-form {
  display: flex; /* 使用Flexbox布局让输入框和按钮在一行 */
  position: absolute; /* 绝对定位，使表单悬浮 */
  bottom: 10px; /* 距离底部的距离，可根据需要调整 */
  left: 10px; /* 距离左侧的距离，可根据需要调整 */
  gap: 5px; /* 项目间的间隔，可自定义 */
  z-index: 2; /* 确保表单在视频之上 */
  gap: 5px; /* 输入框和按钮之间的间距 */
}
.barrage-button {
  font-size: 30px;
  color: #a9895d;
  background: transparent;
  cursor: pointer;
  outline: none;
  border: none;
  border-radius: 5px;
  margin-bottom: 100px;
  margin: 1rem;
  margin-left: 1px;
  padding: 5px 10px;
  text-decoration: underline; /* 添加这一行来显示文字下划线 */
}
.barrage-input {
  flex-grow: 1;
  margin-right: 5px; /* 在输入框和按钮间添加一点间隔 */
  width: 50%;
  height: 55px;
  margin-bottom: 10px;
  outline: none;
  border: none;
  border-bottom: 2px solid #f7debb;
  color: #a9895d;

  font-size: 20px;
  &::-webkit-input-placeholder {
    color: #E8D1B1;
    font-size: 20px;
  }
  &::-moz-placeholder {
    color: #E8D1B1;
    font-size: 20px;
  }
  &:-ms-input-placeholder {
    color: #E8D1B1;
    font-size: 20px;
  }
  &:-moz-placeholder {
    color: #E8D1B1;
    font-size: 20px;
  }
}

.send-button {
  padding: 8px 16px;
  background-color: dodgerblue;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

#main_video {
  position: absolute;
  top: 50%;
  left: 50%;
  min-width: 100%;
  min-height: 100%;
  transform: translate(-50%, -50%);
  object-fit: cover; /* 保持视频宽高比并填充容器 */
}

.wedding-editor{
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  padding: 0px;
  padding-top: 0px;
  overflow-x: hidden;
  overflow-y: auto;
  z-index: 1;
  transform-origin: 0 0;
  -webkit-transform-origin: 0 0;
  transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  -webkit-transition: all 1.6s cubic-bezier(0.4, 0, 1, 1);
  p.code{
    margin: 0;
    color: #BBB;
    line-height: 1.2;
    font-family: 'Roboto Mono', 'Menlo', 'Monaco', Courier, monospace !important;
    font-weight: 500 !important;
    font-size: 16px!important;
  }
  pre{
    margin-top: 40px; /* 调整上边距 */
    white-space: pre-wrap;
    code{
      white-space: pre-wrap;
      word-break: break-all;
      font-size: 17px!important;
      margin: 0; /* 调整代码块与周围内容的间距 */
      color: #ffd69b;
      line-height: 1.2;
      font-family: 'Roboto Mono', 'Menlo', 'Monaco', Courier, monospace !important;
      font-weight: 500 !important;
      background: transparent;
    }
  }
  .inner-text{
    /*文字部分*/
    font-size: 28px;
    display: inline-block;
    vertical-align: bottom;
    margin-top: 10px;
  }
  .editor-open{
    position: fixed;
    right: 20px;
    bottom: 20px;
    width: 60px;
    height: 60px;
    padding: 6px;
    border-radius: 30px;
    text-align: center;
    line-height: 18px;
    border: 5px solid #ffd69b;
    color: #a9895d;
    background: #FFF1DE;
    z-index: 1000;
  }
  .barrage-layer {
    top: 100px;
    position: absolute ;
    pointer-events: none; /* 可选，使得鼠标穿透弹幕，不影响视频下方的交互 */
    z-index: 1; /* 确保弹幕层在视频层之上 */
  }
  .barrage-input {
    position: fixed; /* 改为fixed定位，使其相对于浏览器窗口定位 */
    bottom: 20px; /* 距离底部20px */
    left: 20px; /* 距离左侧20px */
    z-index: 1; /* 确保在视频层之上 */
  }
}
</style>
