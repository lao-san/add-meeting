
<template>
  <div class="mod-config">
    <el-row>
      <el-col :span="6">
        <div class="grid-content bg-purple-light">
          <el-button
            v-if="isAuth('admin:meeting:save')"
            type="primary"
            @click="addOrUpdateHandle()"
          >+ 创建新会议</el-button>
        </div>
      </el-col>
    </el-row>
    <el-row v-if="reFresh">
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>未发布中的会议</h2>
        </div>
      </el-col>
      <meeting-box v-for="item in unpublished" :key="item.id" :list="item" style="margin-top:10px;"></meeting-box>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>发布中的会议</h2>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <meeting-box v-for="item in release" :key="item.id" :list="item" style="margin-top:20px;"></meeting-box>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>已结束的会议</h2>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <meeting-box v-for="item in delRelease" :key="item.id" :list="item" style="margin-top:20px;"></meeting-box>
    </el-row>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "../modules/admin/meeting-add-or-update";
import MeetingBox from "../../components/meetingbox";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [], //获取数据
      unpublished: [], // 未发布
      release: [], //已发布
      delRelease: [], //结束
      pageIndex: 0,
      pageSize: 30,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false,
      meetingId: "",
      reFresh: true
    };
  },
  components: {
    AddOrUpdate,
    MeetingBox
  },
  activated() {},

  created() {
    this.getDataList();
  },

  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/admin/meeting/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
          this.getHandleDataList(data.page.list);
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
      });
    },
    //将获取的数据分三个数组分别为发布/未发布/结束
    getHandleDataList(list) {
      const meetingList = list;
      meetingList.forEach(item => {
        // console.log(item.isCheck)
        switch (item.isCheck) {
          case 0: //未发布
            this.unpublished.push(item);
            break;
          case 1: //已发布
            this.release.push(item);
            break;
          default:
            //已结束
            this.delRelease.push(item);
            break;
        }
      });
    },
    // 每页数
    sizeChangeHandle(val) {
      this.pageSize = val;
      this.pageIndex = 1;
      this.getDataList();
    },
    // 当前页
    currentChangeHandle(val) {
      this.pageIndex = val;
      this.getDataList();
    },
    // 多选
    selectionChangeHandle(val) {
      this.dataListSelections = val;
    },
    // 新增 / 修改
    addOrUpdateHandle(id) {
      this.addOrUpdateVisible = true;
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id);
      });
    },
    // 删除
    deleteHandle(id) {
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
            return item.id;
          });
      this.$confirm(
        `确定对[id=${ids.join(",")}]进行[${id ? "删除" : "批量删除"}]操作?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/admin/meeting/delete"),
          method: "post",
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.$message({
              message: "操作成功",
              type: "success",
              duration: 1500,
              onClose: () => {
                this.getDataList();
              }
            });
          } else {
            this.$message.error(data.msg);
          }
        });
      });
    },
    // 获取会议id
    getMeetingListId(data) {
      this.meetingId = data;
      console.log(this.meetingId);
    }
  }
};
</script>
<style lang="scss" scoped>
</style>>

