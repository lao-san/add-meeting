<template>
  <div class="ll-content">
    <h1>{{meetingData.nameCn}}</h1>
    <div class="box">
      <div class="bgMeeting" :style="{ background: 'url(http://121.42.53.174:9008/static' + meetingData.titlePicture + ') 100% 100%' }"></div>

      <el-form label-width="200px">
        <el-form-item label="会议名称" class="fz_30">{{meetingData.nameCn}}</el-form-item>
        <el-form-item label="会议时间">{{meetingData.startTime}}</el-form-item>
        <el-form-item label="举办地点">{{meetingData.address}}</el-form-item>
        <el-form-item label="涉及学科">
          <el-tag v-for="(item) in subjectsList" :key="item" style="margin-right:10px">{{item}}</el-tag>
        </el-form-item>
        <el-form-item label="涉及行业">
          <el-tag v-for="(item) in industries" :key="item" style="margin-right:10px">{{item}}</el-tag>
        </el-form-item>
        <el-form-item label="会议负责人">{{this.meetingData.serviceEmp}}</el-form-item>
        <el-form-item label="会议介绍">
          <div class="introduction">{{this.meetingData.introduction}}</div>
        </el-form-item>
      </el-form>
      <div class="code"></div>
    </div>
  </div>
</template>
<script>
export default {
  name: "",
  data() {
    return {
      id: "", //会议id
      meetingData: {},
      subjectsList: [],
      industries: []
    };
  },
  components: {},
  computed: {},
  beforeMount() {},
  created() {},
  activated() {
    this.getMeeting();
  },
  methods: {
    getMeeting() {
      this.id = this.$route.params.id;
      if (this.id) {
        this.$http({
          url: this.$http.adornUrl(`/admin/meeting/info/${this.id}`),
          method: "get",
          params: this.$http.adornParams()
        }).then(res => {
          // window.console.log(res);
          if (res.data && res.data.code === 0) {
            this.meetingData = res.data.meeting;
            this.subjectsList = res.data.meeting.subjects.split(",");
            this.industries = res.data.meeting.industries.split(",");
          }
        });
      }
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
      width: 630px;
      height: 300px;
      // background-color: red;
      margin: 0 auto;
    }
    .introduction {
      width: 300px;
    }
  }
}
</style>
