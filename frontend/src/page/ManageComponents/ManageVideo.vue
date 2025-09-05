<template>
  <div class="main planCenterColumn">
    <div class="video-manage-container">
      <div class="waiting-list planCenterColumn">
        <h3>待审列表</h3>
        <div class="video-box-layer planCenterRow">
          <div v-for="video in VideoInterface.inofrom" :key="video" class="video-box-per"
               @mouseover="show_video_marsk(video)"
               @mouseleave="hide_video_marsk(video)"
               @click="click_video_marsk(video)"
               :style="video.color"
          >
            <div v-show="video.focus=='true'" class="video-marsk">
              <img src="../../icon/live.png" alt="" class="video-live">
            </div>
            <img :src="video.imgUrl" alt="" class="video-img">
          </div>
        </div>
      </div>
      <div class="editor-panel planCenterColumn">
        <div class="current-video" v-show="options.src">
          <vue3VideoPlay v-bind="options" :poster="options.imgUrl"/>
        </div>
        <div v-show="!options.src" class="no-video planCenterRow">
          <h2>选择一个视频开始工作吧</h2>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {onMounted, reactive, getCurrentInstance} from 'vue';
import axios from 'axios';
export default ({
  name:"ManageVideo",
  props:{
    op: { type: Object, default: {name:"",func:""} }
  },
  emits:['askForShowConfirmTable'],
  watch:{
    op() {
      switch(this.op.func){
        case "confirm":
          let urlTail = this.$data.pass?"/ContentManagement/Video/VideoReviewedPass":"/ContentManagement/Video/VideoReviewedReject";
          this.postVideoResult(urlTail)
          break
        case "0":
          if(this.$data.selectedVideo==null){
            alert("未选择视频！")
          }else{
            this.$data.pass = true
            this.ctx.emit('askForShowConfirmTable',"视频审核通过")
          }
          break
        case "1":
          if(this.$data.selectedVideo==null){
            alert("未选择视频！")
          }
          else{
            this.$data.pass = false
            this.ctx.emit('askForShowConfirmTable',"视频驳回")
          }
          break
      }
    }
  },
  setup(props,ctx) {
    const cns = getCurrentInstance()
    let url = cns.appContext.config.globalProperties.$url
    let VideoInterface = reactive({
      inofrom: []
    })
    onMounted(async () => {
      if(cns.appContext.config.globalProperties.$user.userName==null){
        let userInfo = JSON.parse(localStorage.getItem('userInfo'));
        cns.appContext.config.globalProperties.$user.userName = userInfo.name
        cns.appContext.config.globalProperties.$user.isLogin = userInfo.isLogin
        cns.appContext.config.globalProperties.$user.token = userInfo.token
      }
      axios.post(url + "/ContentManagement/Video/getJobs", {
        "UserPowerCheck":{
          "id":cns.appContext.config.globalProperties.$user.userName,
          "token":cns.appContext.config.globalProperties.$user.token,
        },
        "Reviewer":cns.appContext.config.globalProperties.$user.userName
      }).catch(function (error) {
        this.$message({
          message: '请求失败,权限不足',
          type: 'error'
        })
      }).then(response => {
        console.log(response)
        if(response.data!=""){
          response.data.data.forEach(element => {
            VideoInterface.inofrom.push({
              id: element.id,
              imgUrl: element.pic_id,
              videoUrl: element.url,
              videoName: element.name,
              upLoadNmae: element.owner_id,
              focus: "false",
              color:"background-color: white;"
            })
          });
        }
      })
    })
    // 视频播放器设置
    const options = reactive({
      width: "800px", //播放器高度
      height: "450px", //播放器高度
      color: "#409eff", //主题色
      title: "", //视频名称
      src: false, //视频源
      muted: false, //静音
      webFullScreen: false,
      imgUrl:null,
      speedRate: ["0.75", "1.0", "1.25", "1.5", "2.0"], //播放倍速
      autoPlay: false, //自动播放
      loop: false, //循环播放
      mirror: false, //镜像画面
      ligthOff: false, //关灯模式
      volume: 0.3, //默认音量大小
      control: true, //是否显示控制
      controlBtns: [
        "audioTrack",
        "quality",
        "speedRate",
        "volume",
        "setting",
        "pip",
        "pageFullScreen",
        "fullScreen",
      ], //显示所有按钮,
    })
    return {
      options,
      VideoInterface,
      ctx
    };
  },
  data(){
    return{
      selectedVideo:null,
      pass:null
    }
  },
  methods:{
    //点击视频播放跳转页面
    click_video_marsk: function (video) {
      for(const element of this.VideoInterface.inofrom){
        element.focus = false
      }
      video.focus = 'true'
      this.$data.selectedVideo = video
      this.options.src = video.videoUrl
      this.options.imgUrl = video.imgUrl
    },
    //鼠标移至视频上出现边框
    show_video_marsk: function (video) {
      video.color = "background-color: rgb(115, 115, 115);"
    },
    //鼠标离开视频上隐藏边框
    hide_video_marsk: function (video) {
      video.color = "background-color: white;"
    },
    postVideoResult:function(urlTail){
      const that = this
      axios({
        url: this.$url + urlTail,
        method: "post",
        data: {
          "UserPowerCheck":{
            "id":this.$user.userName,
            "token":this.$user.token,
          },
          "Reviewer":this.$user.userName,
          "VideoID":this.$data.selectedVideo.id
        },
      }).then((response) => {
        if(response.data.msg!="ok"){
          this.$message({
            message: '提交失败',
            type: 'error'
          })
        }else{
          this.$message({
            message: '提交成功',
            type: 'success'
          })
          let newVideolist = []
          for(const element of that.VideoInterface.inofrom){
            if(element.id!=that.$data.selectedVideo.id)
              newVideolist.push(element)
          }
          that.VideoInterface.inofrom = newVideolist
          that.$data.selectedVideo = null
          that.options.src = false
        }

      });
      this.$message({
        message: '提交中,请等待...',
        type: 'success'
      })
    }
  }
})
</script>

<style scoped>
.main {
  width: 100%;
  height: 100%;
  background-image: url("../../../src/new/imgs/scroll2.png");
  background-size:100% 15%;
  background-position: bottom center;
  background-repeat: no-repeat ;
  background-color: #f3eddc;

}
h3{
  color:rgb(93, 76, 54);
}
.planCenterColumn {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.planCenterRow {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;;
}

/* 左右布局容器与背景配色 */
.video-manage-container{
  width: 80vw;
  height: 70vh;
  margin-top: 30px;
  margin-left: 3vw;
  padding-left: 200px;
  padding-right: 150px;
  padding-top: 16px;
  padding-bottom: 32px;
  display: flex;
  flex-direction: row;
  gap: 25px;
  justify-content: space-between;
  align-items: flex-start;
  background-color: #aa0d05; /* 红色主题背景 */
  //border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.15);
}
/* 每层视频的大小设置 */
.video-box-layer {
  width: 100%;
  height: 18vh;
}
.waiting-list {
  color: black;
  width: 25vw;
  height: 66.2vh;
  background-color: #ffffff; /* 左侧面板背景 */
  padding: 16px;
  //border-radius: 10px;
}
/* 单个视频大小设置 */
.video-box-per {
  width: 12vw;
  height: 18vh;
  margin-right: 1vw;
  background-color: white;
  border-radius: 1vw;
}
/*视频上出现阴影设置*/
.video-marsk {
  width: 12vw;
  height: 16vh;
  border-radius: 1vw;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1;
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
}
/*视频播放图标图片大小设置*/
.video-live {
  width: 15vh;
  height: 15vh;
}
/* 单个视频图片设置 */
.video-img {
  width: 12vw;
  height: 16vh;
  border-radius: 1vw;
}
/* 单个视频名字设置 */
.video-name {
  color: slategray;
  font-size: 0.5em;
}
/* 单个视频作者名字设置 */
.upLoad-name {
  color: black;
  font-size: 0.5em;
}
.editor-panel{
  width: 60vw;
  min-height: 65vh;
  background-color: #ffffff; /* 右侧面板背景 */
  //border-radius: 10px;
  padding: 20px;
}
.current-video {
  width: 100%;
  height: 65vh;
  background: black;
}
.no-video {
  width: 100%;
  height: 65vh;
  background: #faeed5;
  color: #8c6326;
}
</style>