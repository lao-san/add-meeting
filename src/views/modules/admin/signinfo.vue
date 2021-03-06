<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="姓名/负责人" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('admin:signinfo:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >新增</el-button>
        <el-button
          v-if="isAuth('admin:signinfo:delete')"
          type="danger"
          @click="deleteHandle()"
          :disabled="dataListSelections.length <= 0"
        >批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;"
    >
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="truename" header-align="center" align="center" label="姓名"></el-table-column>
      <el-table-column prop="organization" header-align="center" align="center" label="所属机构"></el-table-column>
      <!-- <el-table-column prop="jobTitle" header-align="center" align="center" label="职称"></el-table-column> -->
      <el-table-column prop="phone" header-align="center" align="center" label="电话"></el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="签到状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status===1">已签到</el-tag>
          <el-tag v-else type="danger">未签到</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="badge" header-align="center" align="center" label="是否完成胸卡打印">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.badge===1">是</el-tag>
          <el-tag v-else type="danger">否</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="money" header-align="center" align="center" label="金额(单位元)"></el-table-column>
      <el-table-column prop="servername" header-align="center" align="center" label="负责人"></el-table-column>
      <el-table-column prop="isPay" header-align="center" align="center" label="是否缴费">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.isPay === 0"
            size="medium"
            hit="true"
            @click="changeStatus(1,scope.row.pid)"
          >否</el-button>
          <el-button
            v-else-if="scope.row.isPay === 1"
            size="medium"
            hit="true"
            type="success"
            @click="changeStatus(0,scope.row.pid)"
          >是</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column>
      <el-table-column prop="lastTime" header-align="center" align="center" label="最后签到时间"></el-table-column>
      <el-table-column prop="note" header-align="center" align="center" label="备注"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <!-- <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button> -->
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"
    ></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from "./signinfo-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    };
  },
  components: {
    AddOrUpdate
  },
  activated() {
    this.getDataList();
  },
  methods: {
    // 获取数据列表
    getDataList() {
      this.dataListLoading = true;
      this.$http({
        url: this.$http.adornUrl("/admin/signinfo/listreg"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key,
          meetingId: this.$route.params.id
        })
      }).then(({ data }) => {
        window.console.log(data)
        if (data && data.code === 0) {
          this.dataList = data.page.list;
          this.totalPage = data.page.totalCount;
        } else {
          this.dataList = [];
          this.totalPage = 0;
        }
        this.dataListLoading = false;
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
      this.$confirm(`确定删除?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/admin/signinfo/delete"),
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
    // 修改缴费状态
    changeStatus(status, id) {
      window.console.log(status,id)
      this.$http({
        url: this.$http.adornUrl("/admin/payment/paystatus"),
        method: "post",
        data: this.$http.adornParams({
          id: id,
          status: status
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 500,
            onClose: () => {
              var dList = this.dataList;
              dList.forEach(element => {
                if (element.pid === id) {
                  element.isPay = status;
                }
              });
              this.dataList = dList;
            }
          });
        } else {
          this.$message.error(data.msg);
        }
      });
    }
  }
};
</script>
