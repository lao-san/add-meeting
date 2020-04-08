<template>
  <el-col :span="4" style="margin-left:30px"  @click.native="handClick()" >
    <el-card :body-style="{ padding: '8px' }" style="height:380px">
      <div class="bg" :style="{backgroundImage: 'url(' + httpimage + ')', backgroundSize:'cover'}"></div>
      <div style="padding: 14px;">
        <span style="font-size:20px">{{this.list.nameCn}}</span>
        <div class="bottom clearfix">
          <time class="time">{{this.list.address}}</time>
          <p class="time">举办时间:{{this.list.startTime}}</p>
          <!-- <el-button type="text" class="button">操作按钮</el-button> -->
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
      id:'',
      httpimage:
        "http://a2.att.hudong.com/36/48/19300001357258133412489354717.jpg"
    };
  },
  props: {
    list: Object
  },
  components: {},
  computed: {},
  beforeMount() {},
  mounted() {
    this.getMeetingId()
  },
  methods: {
    handClick(){
      this.$router.push({path:`/admin-meeting/${this.id}`})
    },
    getMeetingId() {
      this.id = this.list.id
    }
  },
  watch: {}
};
</script>
<style scoped lang='scss'>
.time {
  font-size: 13px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
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
