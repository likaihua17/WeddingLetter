<template>
  <div  class="wedding-invitation" :class="{ 'invitation-bounce':canOpen }"  >
    <div class="invitation-container" :class="{ 'invitation-down':isOpening }">
      <div class="invitation-cover">
        <div  class="cover-content" :class="{'invitation-up':isOpening}">
          <div ref="invitation" v-show="isShow" class="content-inside">
            <iframe
                class="content-inside-iframe"
                ref="iframe"
                src="/static/index.html"
                width="100%"
                height="96%"
                frameborder="0"></iframe>
            <p><b> <a :href="boyTelUrl">伍艺平</a> & <a :href="girlTelUrl">余雪丽</a></b></p>
            <p>良辰日：<b>2024年10月04日</b></p>
            <p>良缘府：<b><a @click="showMapChoice = true">
              广安花桥黄记食府
            </a></b>  </p>
            <div class="content-inside-bless">
              <div>
                <button @click="showMapChoice = true">导航前往</button>
                <button @click="closeInvitation">收起</button>
              </div>
            </div>
            <transition name="fade">
              <!-- 弹出模态框 -->
              <div v-if="showMapChoice" class="modal-mask">
                <div class="modal-dialog">
                  <div class="modal-header">
                    <h3>通往幸福之路,您希望谁来为您导航?</h3>
                    <button @click="showMapChoice = false">&times;</button>
                  </div>
                  <div class="modal-body">
                    <button @click="go('bd')">百度地图</button>
                    <button @click="go('gd')">高德地图</button>
                  </div>
                </div>
              </div>
            </transition>
          </div>
        </div>
        <div class="cover-inside-left" :class="{'opening':isOpening}" @click="openInvitation">
          <img  src="../images/qj.png" height="500" width="268">
          <!--<div class="inside-text">
            請柬
          </div>-->
        </div>
        <div class="cover-inside-right" :class="{'opening':isOpening}" @click="openInvitation"></div >
        <img class="cover-inside-seal" src="../images/seal.png" @click="openInvitation"
             :class="{'invitation-flight':isOpening}">
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['canOpen'],
  data() {
    return {
      //新郎名称
      boyName:'',
      //新娘
      girlName:'',

      mapGdUrl: 'https://uri.amap.com/marker?position=106.694152,30.78266&name=广安花桥黄记食府-伍艺平余雪丽诚挚邀请',
      mapBdUrl: 'https://api.map.baidu.com/marker?location=106.694152,30.78266&title=广安花桥黄记食府&content=伍艺平余雪丽诚挚邀请&output=html&src=webapp.reformer.appname&coord_type=gcj02',
      boyTelUrl: 'tel:15520131483',
      girlTelUrl: 'tel:15520131485',
      isOpening: false,
      wish: '',
      isFocused: false,
      showMapChoice: false,
      hasEntered: false,
      isShow:false,
    }
  },
  methods: {
    go(select) {
      switch (select) {
        case 'gd':
          window.location.href = this.mapGdUrl;
          break;
        case 'bd':
          window.location.href = this.mapBdUrl;
          break;
        default:
          window.location.href = this.mapGdUrl;
          break;
      }
    },
    // 打开邀请函
    openInvitation() {
      // 保持页面一直滚到最下面
      if(this.isOpening){//如果当前是打开
        return
      }
      document.querySelector('iframe').contentWindow.location.reload();
      this.isShow=true
      //刷新页面  展开一部
      //播放音乐
      this.$emit('onOpen')
      this.isOpening = true
      this.$nextTick(() => {
        this.$refs.invitation.scrollTop = 0;
      });
    },
    closeInvitation() {
      this.isShow = false
      this.isOpening = false
      setTimeout(() => {
        this.$emit('onClose')
      }, 660)
    },
    // 发送弹幕
    sendBarrage() {
      this.$nextTick(() => {
        this.hasEntered = true
        if (!this.wish) {
          return
        }
        window.location.href = 'sms:15520131485?body='+encodeURIComponent(this.wish);
        this.isOpening = false
        this.$refs.wishInput.blur()
        setTimeout(() => {
          this.$emit('sendBarrage', this.wish)
        }, 660)
      })
    }
  }

}
</script>

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

    /*邀请函的上下位置*/
    &.invitation-down {
      transform: translateY(-10px);
      -webkit-transform: translateY(-10px);
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
        /*里面网页的离底部的位置*/
        top: 60px;
        left: 0;
        width: 100%;
        height: calc(100% - 0px);
        padding: 10px 20px;
        transition: transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        -webkit-transition: -webkit-transform 0.6s cubic-bezier(0.4, 0, 1, 1);
        &.invitation-up {
          transform: translateY(-60px);
          -webkit-transform: translateY(-60px);
        }
        .content-inside {
          height: 100%;
          padding: 5px;
          color: rgba(88, 128, 187, 1);
          background-color: #fff;
          text-align: center;
          overflow: auto;
          .content-inside-iframe {
            width: 95%;
            /*内嵌页面的高度和宽度*/
            height: 80%;
            margin-bottom: 0px;
            padding: 5px;
            border: 1px solid #dbdbdb;
          }
          p {
            margin-top: 0;
            margin-bottom: 5px;
          }

          .content-inside-bless {
            input {
              width: 100%;
              height: 35px;
              margin-bottom: 10px;
              outline: none;
              border: none;
              border-bottom: 1px solid #f7debb;
              color: #a9895d;
              background: transparent;
              font-size: 16px;

              &::-webkit-input-placeholder {
                color: #E8D1B1;
                font-size: 12px;
              }

              &::-moz-placeholder {
                color: #E8D1B1;
                font-size: 12px;
              }

              &:-ms-input-placeholder {
                color: #E8D1B1;
                font-size: 12px;
              }

              &:-moz-placeholder {
                color: #E8D1B1;
                font-size: 12px;
              }
            }
            > div {
              display: flex;
              button {
                // width: 100%;
                height: 35px;
                color: #a9895d;
                background: #f7debb;
                border: none;
                outline: none;
                &:disabled {
                  opacity: 0.8;
                }
                &:first-child {
                  margin-right: 10px;
                  flex: 1;
                }
                &:last-child {
                  width: 60px;
                  border: 1px solid #f7debb;
                  background: transparent;
                }
              }
            }
          }
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
        transition: transform 0.5s;
        -webkit-transition: -webkit-transform 0.5s;
        transform-origin: 0 50%;
        -webkit-transform-origin: 0 50%;
        &.opening {
          transform: rotate3d(0, 1, 0, -140deg);
          -webkit-transform: rotate3d(0, 1, 0, -140deg);
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
        transition: transform 0.5s;
        -webkit-transition: -webkit-transform 0.5s;
        transform-origin: 100% 50%;
        -webkit-transform-origin: 100% 50%;
        &.opening {
          transform: rotate3d(0, 1, 0, 140deg);
          -webkit-transform: rotate3d(0, 1, 0, 140deg);
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
  min-width: 250px;
  max-width: 350px;
  min-height: 150px;
  max-height: 200px;
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
