<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="订单编号/付款人" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <!-- <el-button
          v-if="isAuth('admin:payment:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >新增</el-button>-->
        <el-button
          v-if="isAuth('admin:payment:delete')"
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
      <el-table-column prop="orderId" header-align="center" align="center" label="订单号"></el-table-column>
      <el-table-column prop="truename" header-align="center" align="center" label="缴费人"></el-table-column>
      <el-table-column prop="organization" header-align="center" align="center" label="所在机构"></el-table-column>
      <!-- <el-table-column prop="feeId" header-align="center" align="center" label="费用类型id"></el-table-column> -->
      <el-table-column prop="payType" header-align="center" align="center" label="缴费方式">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.payType===1" type="success">线上缴费</el-tag>
          <el-tag v-else="scope.row.payType===2">现场缴费</el-tag>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="payOption" header-align="center" align="center" label="缴费项目"></el-table-column> -->
      <el-table-column prop="isPay" header-align="center" align="center" label="是否付费">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.isPay===0" type="danger">否</el-tag>
          <el-tag v-else="scope.row.isPay===1">是</el-tag>
        </template>
      </el-table-column>
      <!-- <el-table-column prop="taxTitle" header-align="center" align="center" label="发票抬头"></el-table-column>
      <el-table-column prop="taxNumber" header-align="center" align="center" label="纳税人税号"></el-table-column>-->
      <el-table-column prop="phone" header-align="center" align="center" label="联系电话"></el-table-column>
      <!-- <el-table-column prop="bankAddrAccount" header-align="center" align="center" label="开户行及账号"></el-table-column> -->
      <!-- <el-table-column prop="taxAddress" header-align="center" align="center" label="邮寄地址"></el-table-column> -->
      <!-- <el-table-column prop="isCheck" header-align="center" align="center" label="确认是否已缴费"></el-table-column>
      <el-table-column prop="taxType" header-align="center" align="center" label="发票类型0普1专"></el-table-column>-->
      <el-table-column prop="payTime" header-align="center" align="center" label="缴费时间"></el-table-column>

      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
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
import AddOrUpdate from "./payment-add-or-update";
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
        url: this.$http.adornUrl("/admin/payment/list"),
        method: "get",
        params: this.$http.adornParams({
          meetingId: this.$route.params.id,
          page: this.pageIndex,
          limit: this.pageSize,
          key: this.dataForm.key
        })
      }).then(({ data }) => {
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
      this.$confirm(
        `确定删除?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl("/admin/payment/delete"),
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
    handClick(id) {
      window.console.log(id);
    }
  }
};
</script>
