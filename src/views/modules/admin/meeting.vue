<template>
  <div class="ll-content" v-loading="loading">
    <h1 >{{meetingData.nameCn}}</h1>
    <div class="box">
      <img :src="imageNull" class="bgMeeting" />
      <el-form label-width="200px" style="margin-top:20px">
        <!-- <el-form-item label="会议名称" class="fz_30" style="margin-top:50px">{{meetingData.nameCn}}</el-form-item> -->
        <el-form-item label="会议时间">{{meetingData.startTime}}</el-form-item>
        <el-form-item label="举办地点">{{meetingData.address}}</el-form-item>
        <el-form-item label="涉及学科">
          <el-tag v-for="(item) in subjectsList" :key="item" style="margin-right:10px">{{item}}</el-tag>
        </el-form-item>
        <el-form-item label="涉及行业">
          <el-tag v-for="(item) in industries" :key="item" style="margin-right:10px">{{item}}</el-tag>
        </el-form-item>
        <el-form-item label="会议负责人">{{this.meetingData.serviceName}}</el-form-item>
        <el-form-item label="会议介绍">
          <div class="introduction">{{this.meetingData.introduction}}</div>
        </el-form-item>
      </el-form>
      <div class="code"></div>
    </div>
  </div>
</template>
<script>
import { set } from "@/utils";

export default {
  name: "",
  data() {
    return {
      id: "", //会议id
      meetingData: {},
      subjectsList: [],
      industries: [],
      image: "",
      loading: false
    };
  },
  components: {},
  computed: {
    imageNull() {
      return this.image.includes(",")
        ? this.imageUrl +
            this.image
              .split(",")
              .splice(0, 1)
              .join()
        : this.imageUrl + this.image;
    }
  },
  beforeMount() {},
  created() {
    this.demo()
  },
  activated() {
    this.getMeeting();
  },
  methods: {
    getMeeting() {
      this.loading = true;
      this.id = this.$route.params.id;
      if (this.id) {
        this.$http({
          url: this.$http.adornUrl(`/admin/meeting/info/${this.id}`),
          method: "get",
          params: this.$http.adornParams()
        }).then(res => {
          if (res.data && res.data.code === 0) {
            window.console.log(res);
            this.meetingData = res.data.meeting;
            this.industries = res.data.meeting.industries.split(",");
            // this.industries = set(this.industries);
            this.image = res.data.meeting.titlePicture;
            this.subjectsList = res.data.meeting.subjects.split(",");
            // this.subjectsList = set(this.subjectsList);
            this.loading = false;
          }
        });
      }
    },
    demo(){
      let ll = set(this.subjectsList)
      window.console.log(ll)
    }
  }
};
</script>
<style scoped lang='scss'>
.ll-content {
  height: 100%;
  width: 100%;
  .box {
    position: relative;
    height: 700px;
    width: 900px;
    margin: 0 auto;
    // border: 1px solid red;
    .bgMeeting {
      display: block;
      width: 630px;
      height: 300px;
      // background-color: red;
      margin: 0 auto;
    }
    .introduction {
      width: 400px;
    }
  }
}
</style>
