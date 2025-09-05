<template>
  <div class="main">
    <el-card v-for="page in pages.page" :key="page">
      <div>
        <h2 class="main-text">{{ page.title }}</h2>
        <div class="main-text">
          {{ page.content }}
        </div>
      </div>
      <div class="article-footer">
        <i class="article-icon">{{ page.author }}</i>
        <i class="el-icon-date article-icon">{{ page.time }}</i>
      </div>
    </el-card>
  </div>
</template>


<script>
import {onMounted, reactive, ref, getCurrentInstance} from 'vue';
import axios from "axios";
export default {
  name: "upload",
  setup() {
      const cns = getCurrentInstance();
      let url = cns.appContext.config.globalProperties.$url;
      let pages = reactive({
      page: []
    })
      onMounted(async () => {
      axios.post(url + "/PersonalArticles/getPersonalArticles",{}).then(response => {
          pages.page = response.data.data
      }
      )
    }
    )
      return {pages};
    },
  data() {
    return {
      showDialog: false,
    };
  },
  methods: {
  },
  created() {},
};
</script>

<style scoped>
.main {
  margin: 0 auto;                /* 居中 */
  padding: 20px;
  margin-top: 80px;
  max-width: 1200px;             /* 限制整体宽度 */
  background-color: #f9f9f9;     /* 背景淡灰色 */
  min-height: 100vh;
  box-sizing: border-box;        /* 避免padding撑开宽度 */
}

.el-card {
  margin-top: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  position: relative;            /* 方便放置右下角元素 */
  padding-bottom: 40px;          /* 给作者/时间留空间 */
}

.el-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 18px rgba(0,0,0,0.2);
}

.main-text {
  font-size: 18px;
  color: #333;
  line-height: 1.8;
  margin-bottom: 12px;
}

.main-text h2,
h2.main-text {
  font-size: 24px;
  font-weight: bold;
  color: #2c3e50;
  margin-bottom: 10px;
}

.article-footer {
  position: absolute;
  right: 15px;
  bottom: 10px;
  display: flex;
  gap: 12px;
}

.article-icon {
  color: #666;
  font-size: 14px;
}

.article-icon:first-of-type {
  font-weight: 500;
  color: #2980b9;
}

.el-icon-date.article-icon {
  color: #888;
}


</style>
