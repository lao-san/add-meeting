<template>
  <el-col :span="4" style="margin-left:30px">
    <el-card :body-style="{ padding: '8px' }" style="height:400px">
      <div
        class="bg"
        :style="{backgroundImage: 'url( ' + image + ')', backgroundSize:'cover'}"
        @click="handClick()"
      ></div>
      <div style="padding: 14px;">
        <span style="font-size:20px">{{this.list.nameCn}}</span>
        <div class="bottom clearfix">
          <time class="time">{{this.list.address}}</time>
          <p class="time">举办时间:{{this.list.startTime}}</p>
          <el-button
            type="text"
            class="button"
            v-if="this.list.isCheck===0"
            @click="status"
            style="margin-right:50px"
          >发布</el-button>
          <el-button type="text" class="button" v-if="this.list.isCheck===1" @click="status">结束会议</el-button>
          <el-button
            type="text"
            class="button"
            v-if="this.list.isCheck===0||this.list.isCheck===2"
            style="color:red"
            @click="delMeeting"
          >删除</el-button>
          <!-- <el-button type="text" class="button" @click="handClick" style="margin-right:150px">传值</el-button> -->
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
      image: null,
      imageUrl: this.imageUrl
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
        this.image = this.imageUrl + this.image;
      } else {
        this.image = this.httpimage;
        this.image = this.imageUrl + this.image;
      }
    },
    //修改状态
    status() {
      this.$http({
        url: this.$http.adornUrl("/admin/meeting/status"),
        method: "post",
        data: {
          id: this.list.id,
          status: this.list.isCheck === 0 ? 1 : 2
        }
      }).then(({ data }) => {
        window.console.log(data);

        if (data) {
          this.$emit("statusMeeting");
        } else {
          window.console.log(data);
        }
      });
    },
    //删除
    delMeeting() {
      let arr = [];
      arr[0] = this.list.id;
      this.$http({
        url: this.$http.adornUrl("/admin/meeting/delete"),
        method: "post",
        data: this.$http.adornData(arr, false)
      })
        .then(res => {
          window.console.log(res);
          this.$emit("statusMeeting");
        })
        .catch(err => {
          window.console.log(err);
        });
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
