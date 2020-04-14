<template>
  <el-col :span="4" style="margin-left:30px">
    <el-card :body-style="{ padding: '8px' }" style="height:400px">
      <div
        class="bg"
        :style="{backgroundImage: 'url( http://121.42.53.174:9008/static' + image + ')', backgroundSize:'cover'}"
        @click="handClick()"
      ></div>
      <div style="padding: 14px;">
        <span style="font-size:20px">{{this.list.nameCn}}</span>
        <div class="bottom clearfix">
          <time class="time">{{this.list.address}}</time>
          <p class="time">举办时间:{{this.list.startTime}}</p>
          <el-button type="text" class="button" v-if="this.list.isCheck===0">发布</el-button>
          <el-button type="text" class="button" v-if="this.list.isCheck===1">会议结束</el-button>
          <el-button type="text" class="button" v-if="this.list.isCheck===2">会议结束</el-button>
        </div>
      </div>
    </el-card>
  </el-col>
</template>
<script>
export default {
  name: "",
  data() {
    return {
      id: "",
      httpimage: "",
      image: null
    };
  },
  props: {
    list: Object
  },
  components: {},
  computed: {
    imageNull() {}
  },
  beforeMount() {},
  mounted() {
    this.getMeetingId();
  },
  methods: {
    handClick() {
      this.$router.push({ path: `/admin-meeting/${this.id}` });
    },
    getMeetingId() {
      this.id = this.list.id;
      this.httpimage = this.list.titlePicture;
      if (this.httpimage.includes(",")) {
        this.image = this.httpimage
          .split(",")
          .splice(0, 1)
          .join();
      } else {
        this.image = this.httpimage;
      }
    }
  },
  watch: {}
};
</script>
<style scoped lang='scss'>
.el-card {
  position: relative;
}

.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  position: absolute;
  bottom: -2px;
  right: 15px;
  padding-top: 0px;
  float: right;
}

.image {
  width: 100%;
  display: block;
}
.bg {
  width: 100%;
  height: 250px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}
</style>
