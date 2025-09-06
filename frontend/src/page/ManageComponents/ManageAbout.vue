<template>
  <div class="main">
    <div class="topic planCenterColumn">
      <div class="topic_setting">
        主题设置
      </div>
      <div class="writing">
        <div class="planCenterColumn">
          <div>主题</div>
          <input type="text" v-model="this.$data.theme">
          <div>标题</div>
          <input type="text" v-model="this.$data.title">
        </div>
        <div class="planCenterColumn">
          <div>内容</div>
          <textarea  class="text1" name="" id="" cols="100" rows="10" v-model="this.$data.content"></textarea>
        </div>
        <div class="planCenterRow">
          <div class="imgTip">背景图</div>
          <input type="file" accept="image/*" @change="changeTitleImg($event,null,'background')">
        </div>
      </div>
    </div>
    <div class="content planCenterColumn">
      <div class="writing">
        <div v-for="content in this.$data.contentList" :key="content" class="planCenterColumn">
          <div class="topic_setting">内容编号{{content.number}}</div>
          <div class="planCenterRow">
            <div class="imgTip">图片一</div>
            <input type="file" accept="image/*" @change="changeTitleImg($event,content,'img1')">
          </div>
          <div class="planCenterRow">
            <div class="imgTip">图片二</div>
            <input type="file" accept="image/*" @change="changeTitleImg($event,content,'img2')">
          </div>
          <div class="planCenterRow">
            <div class="imgTip">图片三</div>
            <input type="file" accept="image/*" @change="changeTitleImg($event,content,'img3')">
          </div>
          <div class="planCenterColumn">
            <div>介绍</div>
            <textarea class="text2" name="" id="" cols="100" rows="10" v-model="content.content"></textarea>
          </div>
        </div>
      </div>
    </div>
    <div class="addContent">
      <div class="addContent-button-container">
        <div class="addContent-button"
             @click="addContent"
             @mousemove="focusAdd('增加')"
             @mouseleave="blurAdd('增加')"
             :style="this.$data.addContentColor"
        >
          增加内容
        </div>
      </div>
      <div class="addContent-button-container">
        <div class="addContent-button"
             @click="deleteContent"
             @mousemove="focusAdd('减少')"
             @mouseleave="blurAdd('减少')"
             :style="this.$data.deleteContentColor">减少内容
        </div>
      </div>
    </div>
  </div>
</template>

<script>
//主题、内容、背景图片          图片1~3 介绍
import axios from "axios";
import { formEmits } from "element-plus";
export default ({
  name:"MamanageAbout",
  data(){
    return{
      theme:"",
      title:"",
      topicImg:"",
      contentList:[
        {img1:"",img2:"",img3:"",content:"",number:1},
        {img1:"",img2:"",img3:"",content:"",number:2},
      ],
      content:"",
      addContentColor:"background-color: whitesmoke;",
      deleteContentColor:"background-color: whitesmoke;",
    }
  },
  props:{
    op: { type: Object, default: {name:"",func:""} }
  },
  emits:['askForShowConfirmTable'],
  setup(props,ctx){
    return{
      ctx
    }
  },
  watch:{
    op() {
      switch(this.op.func){
        case "confirm":
          let uploadContentList = []
          let numbers = []
          let uploadImnFiles = []
          const that = this
          for (const element of this.$data.contentList) {
            uploadContentList.push(element.content)
            numbers.push(element.number)
            uploadImnFiles.push(element.img1[0])
            uploadImnFiles.push(element.img2[0])
            uploadImnFiles.push(element.img3[0])
          }
          axios({
            url: this.$url + "/ContentManagement/About",
            method: "post",
            headers: {
              "Content-Type": "multipart/form-data",
            },
            data: {
              "id":this.$user.userName,
              "token":this.$user.token,
              "theme":this.$data.theme,
              "title":this.$data.title,
              "content":this.$data.content,
              "backImg":this.$data.topicImg[0],
              "numbers":numbers,
              "contentList":uploadContentList,
              "imgFiles":uploadImnFiles
            },
          }).then((response) => {
            console.log(response.data);
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
              that.clear()
            }
          });
          this.$message({
            message: '提交中,请等待...',
            type: 'success'
          })
          break
        case "1":
          if(this.check())
            this.ctx.emit('askForShowConfirmTable',"提交关于界面")
          break
        case "0":
          this.clear()
          break
      }
    },
  },
  methods : {
    changeTitleImg:function(event,obj,chose){
      console.log(event.target.files)
      switch(chose){
        case "img1":
          obj.img1 = event.target.files
          break
        case "img2":
          obj.img2 = event.target.files
          break
        case "img3":
          obj.img3 = event.target.files
          break
        case "background":
          this.$data.topicImg = event.target.files
          break
      }
    },
    clear:function(){
      this.$data.theme = ""
      this.$data.title = ""
      this.$data.content = ""
      this.$data.topicImg = ""
      this.$data.contentList = [
        {img1:"",img2:"",img3:"",content:"",number:1},
        {img1:"",img2:"",img3:"",content:"",number:2},
      ]
    },
    check:function(){
      if(this.$user.userName==null){
        alert("登录已过期")
        return false
      }
      if(this.$data.theme==""){
        alert("请填写主题")
        return false
      }
      if(this.$data.title==""){
        alert("请填写标题")
        return false
      }
      if(this.$data.content==""){
        alert("请填写内容")
        return false
      }
      if(this.$data.topicImg==""){
        alert("请填选择背景图片")
        return false
      }
      for(const element of this.$data.contentList){
        let info = ""
        if(element.img1==""){
          info = "图片一"
        }else if(element.img2=="") {
          info = "图片二"
        }else if(element.img3=="") {
          info = "图片三"
        }else if(element.content=="") {
          info = "介绍"
        }
        if(info!=""){
          alert("请填写内容编号为"+element.number+"的"+info)
          return false
        }
      }
      return true
    },
    addContent:function(){
      this.$data.contentList.push({img1:"",img2:"",img3:"",content:"",number:this.$data.contentList.length+1})
    },
    deleteContent:function(){
      if(this.$data.contentList.length>2){
        this.$data.contentList = this.$data.contentList.slice(0,this.$data.contentList.length-1)
      }
      else{
        alert("内容至少为两项")
      }
    },
    focusAdd:function(name){
      if(name=='增加'){
        this.$data.addContentColor = "background-color: rgb(204, 204, 204);"
      }else{
        this.$data.deleteContentColor = "background-color: rgb(204, 204, 204);"
      }
    },
    blurAdd:function(name){
      if(name=='增加'){
        this.$data.addContentColor = "background-color: whitesmoke;"
      }else{
        this.$data.deleteContentColor = "background-color: whitesmoke;"
      }
    }
  }
})
</script>

<style scoped>
/* 【核心风格：又红又专 - 主色#f4eede（浅底）、#891e18（红）】 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  color: #6b1713; /* 全局文字色：红色主色 */
  font-family: "Microsoft YaHei", "Heiti SC", sans-serif; /* 方正字体，契合红色风格 */
}

/* 主容器：整体布局与背景 */
.main {
  width: 100%;
  height: 90vh;
  padding: 30px 50px;
  margin-left: 0; /* 移除原有左偏移，优化居中效果 */
  background-color: #f4eede; /* 浅底色：奠定主题基调 */
  background-image: url("../../../src/new/imgs/scroll2.png");
  background-size: 100% 12%;
  background-position: bottom center;
  background-repeat: no-repeat;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px; /* 区域间距，提升页面呼吸感 */
  overflow-y: scroll;
}

/* 区域标题栏（主题配置/详情内容配置） */
.section-header {
  width: 100%;
  height: 40px;
  line-height: 40px;
  background-color: #891e18; /* 红色标题栏，强化主题 */
  color: #f4eede; /* 标题文字反色，提升对比度 */
  font-size: 18px;
  font-weight: bold;
  padding-left: 20px;
  border-radius: 4px 4px 0 0; /* 配合容器圆角，避免棱角 */
}
.topic_setting{
  width: 100%;
  background-color: #891e18;
  color:#f4eede;
  font-size: 22px;
  padding: 20px;
}
/* 主题配置区域容器 */
.writing{
  width: 70vw; /* 适配不同屏幕宽度 */
  min-height: 40vh;
  height: auto;
  margin-top: 0;
  margin-right: 0;
  background: #ffffff;
  padding: 20px; /* 内部间距 */
  border-radius: 4px; /* 圆角优化，提升精致度 */
  box-shadow: 0 4px 16px rgba(137, 30, 24, 0.15); /* 红色系阴影，强化主题氛围 */
}

.img-preview img {
  width: 100%;
  height: 100%;
  object-fit: cover; /* 保持图片比例，避免拉伸 */
}

/* 操作按钮容器（增加/减少内容） */
.addContent {
  display: flex;
  flex-direction: row;
  width: 60vw; /* 适配宽度，避免过宽 */
  margin-top: 0;
  margin-bottom: 30px;
  height: 60vh;
  background-color: #891e18; /* 红色背景，强化主题 */
  padding: 18px;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(137, 30, 24, 0.2);
  gap: 20px; /* 两个按钮间距 */

}

/* 按钮容器 */
.addContent-button-container {
  width: 100%;
  height: 60px;
}

/* 操作按钮样式 */
.addContent-button {
  width: 100%; /* 按钮占满容器 */
  height: 100%;/* 限制最大高度，避免变形 */
  background-color: #ffffff;
  color: #891e18;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  cursor: pointer;
  font-size: 16px;
  transition: all 0.3s; /* 配合JS hover效果，提升流畅度 */
  margin: 0;
  justify-content: center;
  text-align: center;
  align-content: center;
}
.addContent-button:hover {
  box-shadow: 0 4px 12px rgba(137, 30, 24, 0.15); /* hover时加深阴影 */
}

/* 按钮图标样式 */
.btn-icon {
  margin-right: 8px;
  font-weight: bold;
}

/* 弹性布局基础类（保留原逻辑，优化兼容性） */
.planCenterColumn {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-top: 1vh;
}
.planCenterRow {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap:10px;
  margin-top:15px;
}

.text1{
  margin-bottom: 10px;
}
.text2{
  margin-top: 15px;
  margin-bottom: 20px;
}
/* 响应式适配：小屏幕（手机/平板） */
@media (max-width: 768px) {
  .main {
    padding: 20px 15px;
  }
  .topic {
    width: 95vw;
  }
  .content-container {
    width: 95vw;
  }
  .addContent {
    width: 90vw;
    flex-direction: column; /* 小屏幕按钮纵向排列 */
  }
  .addContent-button-contaner {
    width: 100%;
  }
  .img-upload-group {
    flex-direction: column; /* 小屏幕图片上传项纵向排列 */
  }
}
</style>