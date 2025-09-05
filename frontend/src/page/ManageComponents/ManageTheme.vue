<template>
  <div class="main">
    <!-- 主题设置区域 -->
    <div class="topic planCenterColumn">
      <div class="topic_setting">主题设置</div>
      <div class="writing">
        <div class="planCenterColumn">
          <div>主题</div>
          <input type="text" class="topic_text" v-model="Theme">
        </div>
      </div>
    </div>

    <!-- 轮播图设置区域 -->
    <div class="carousel-container planCenterColumn">
      <div class="topic_setting">轮播图设置</div>
      <div class="writing">
        <div class="planCenterColumn">
          <div>选择轮播图</div>
          <div class="planCenterRow" style="flex-wrap: wrap; gap: 20px; margin-top: 20px;">
            <div v-for="Chart in this.$data.carouselChart" :key="Chart" class="planCenterRow chose-chart">
              <div class="imgTip">{{Chart.name}}</div>
              <input type="file" accept="image/*" @change="changeChart($event,Chart)" class="input-file">
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 背景图设置区域 -->
    <div class="background-container planCenterColumn">
      <div class="topic_setting">背景图设置</div>
      <div class="writing">
        <div class="planCenterColumn">
          <div v-for="background in this.$data.backgrounds" :key="background" class="planCenterRow chose-file">
            <div class="imgTip">{{background.name}}</div>
            <input type="file" accept="image/*" @change="changeChart($event,background)" class="input-file">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default ({
  name: "ManageTheme",
  props: {
    op: { type: Object, default: { name: "", func: "" } }
  },
  emits: ['askForShowConfirmTable'],
  setup(props, ctx) {
    return {
      ctx
    }
  },
  watch: {
    op() {
      switch (this.op.func) {
        case "confirm":
          let carouselChartFiles = []
          let backgroundsFiles = []
          for (const element of this.$data.carouselChart) {
            carouselChartFiles.push(element.file[0])
          }
          for (const element of this.$data.backgrounds) {
            backgroundsFiles.push(element.file[0])
          }
          const that = this
          axios({
            url: this.$url + "/ContentManagement/Theme",
            method: "post",
            headers: {
              "Content-Type": "multipart/form-data",
            },
            data: {
              "id": this.$user.userName,
              "token": this.$user.token,
              "theme": this.$data.Theme,
              "carouselChartFiles": carouselChartFiles,
              "backgroundsFiles": backgroundsFiles,
            },
          }).then((response) => {
            if (response.data.msg != "ok") {
              this.$message({
                message: '提交失败',
                type: 'error'
              })
            } else {
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
          if (this.check())
            this.ctx.emit('askForShowConfirmTable', "提交主题")
          break
        case "0":
          this.clear()
          break
      }
    },
  },
  data() {
    return {
      Theme: "",
      carouselChart: [
        { name: "图一", file: "" },
        { name: "图二", file: "" },
        { name: "图三", file: "" },
        { name: "图四", file: "" },
        { name: "图五", file: "" }
      ],
      backgrounds: [
        { name: "选择主页背景", file: "" },
        { name: "推荐界面背景", file: "" },
        { name: "视频播放背景", file: "" }
      ]
    }
  },
  methods: {
    changeChart: function (event, Chart) {
      Chart.file = event.target.files
    },
    clear: function () {
      this.$data.Theme = ""
      this.$data.carouselChart = [
        { name: "图一", file: "" },
        { name: "图二", file: "" },
        { name: "图三", file: "" },
        { name: "图四", file: "" },
        { name: "图五", file: "" }
      ]
      this.$data.backgrounds = [
        { name: "选择主页背景", file: "" },
        { name: "推荐界面背景", file: "" },
        { name: "视频播放背景", file: "" }
      ]
    },
    check: function () {
      if (this.$user.userName == null) {
        alert("登录已过期")
        return false
      }
      if (this.$data.Theme == "") {
        alert("请填写主题")
        return false
      }
      for (const element of this.$data.carouselChart) {
        if (element.file == "") {
          alert("未选择" + element.name)
          return false
        }
      }
      for (const element of this.$data.backgrounds) {
        if (element.file == "") {
          alert("未选择" + element.name)
          return false
        }
      }
      return true
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
.topic_text{
  margin-top: 10px;
}
/* 主容器：整体布局与背景（完全对齐目标页面） */
.main {
  width: 85vw;
  height: 90vh;
  margin-left: 10vw;
  padding: 30px 50px;
  background-color: #f4eede; /* 浅底色：奠定主题基调 */
  background-image: url("../../../src/new/imgs/scroll2.png"); /* 复用目标页面底部卷轴背景 */
  background-size: 100% 12%;
  background-position: bottom center;
  background-repeat: no-repeat;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 30px; /* 区域间距，提升页面呼吸感 */
  overflow-y: scroll; /* 滚动适配长内容 */
}

/* 区域标题栏（主题/轮播图/背景图设置）- 复用目标页面红色标题样式 */
.topic_setting {
  width: 70vw;
  background-color: #891e18; /* 红色标题栏，强化主题 */
  color: #f4eede; /* 标题文字反色，提升对比度 */
  font-size: 22px;
  padding: 20px;
}

/* 内容容器（白色卡片）- 完全对齐目标页面样式 */
.writing {
  width: 70vw; /* 适配不同屏幕宽度 */
  min-height: 20vh;
  height: auto;
  background: #ffffff;
  padding: 20px; /* 内部间距 */
  border-radius: 4px; /* 圆角优化，提升精致度 */
  box-shadow: 0 4px 16px rgba(137, 30, 24, 0.15); /* 红色系阴影，强化主题氛围 */
}

/* 图片上传标签样式 - 对齐目标页面"背景图"标签 */
.imgTip {
  width: 120px; /* 固定宽度，保证布局整齐 */
  text-align: right;
  margin-right: 10px;
  font-size: 16px;
}

/* 弹性布局基础类（复用目标页面逻辑，保证一致性） */
.planCenterColumn {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-top: 1vh;
  width: 100%; /* 确保列布局占满容器 */
}

.planCenterRow {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin-top: 15px;
  width: 100%; /* 确保行布局占满容器 */
}

/* 输入框样式优化（对齐目标页面输入框视觉） */
input[type="text"],
.input-file {
  height: 36px;
  padding: 0 12px;
  //border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  color: #6b1713; /* 输入框文字红色系 */
  outline: none;
  transition: border-color 0.3s;
}

input[type="text"] {
  width: 300px; /* 主题输入框固定宽度 */
}

.input-file {
  width: 400px; /* 文件输入框宽度，适配内容 */
  cursor: pointer;
}

/* 输入框聚焦样式（红色边框强化主题） */
input[type="text"]:focus,
.input-file:focus {
  border-color: #891e18;
  box-shadow: 0 0 0 2px rgba(137, 30, 24, 0.1);
}

/* 轮播图/背景图选择项样式优化 */
.chose-chart,
.chose-file {
  justify-content: flex-start; /* 左对齐，避免内容拥挤 */
  padding: 8px 0;
}

/* 响应式适配（复用目标页面逻辑，保证移动端体验） */
@media (max-width: 768px) {
  .main {
    padding: 20px 15px;
  }

  .writing {
    width: 95vw; /* 小屏幕占满宽度 */
  }

  .planCenterRow {
    flex-direction: column; /* 小屏幕纵向排列，避免溢出 */
    align-items: flex-start;
  }

  .imgTip {
    width: 100%;
    text-align: left;
    margin-right: 0;
    margin-bottom: 5px;
  }

  input[type="text"] {
    width: 100%; /* 小屏幕输入框占满宽度 */
  }

  .input-file {
    width: 100%; /* 小屏幕文件输入框占满宽度 */
  }
}
</style>