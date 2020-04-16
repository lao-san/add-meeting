<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button
          v-if="isAuth('admin:attenders:save')"
          type="primary"
          @click="addOrUpdateHandle()"
        >新增</el-button>
        <el-button
          v-if="isAuth('admin:attenders:delete')"
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
      <el-table-column prop="name" header-align="center" align="center" label="姓名"></el-table-column>
      <el-table-column prop="organization" header-align="center" align="center" label="所属机构"></el-table-column>
      <!-- <el-table-column prop="position" header-align="center" align="center" label="职位"></el-table-column>
      <el-table-column prop="jobTitle" header-align="center" align="center" label="职称"></el-table-column>-->
      <!-- <el-table-column prop="phone" header-align="center" align="center" label="电话"></el-table-column> -->
      <!-- <el-table-column prop="email" header-align="center" align="center" label="邮箱"></el-table-column> -->
      <!-- <el-table-column prop="officephone" header-align="center" align="center" label="办公电话"></el-table-column> -->
      <!-- <el-table-column prop="intention" header-align="center" align="center" label="参会意向">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.intention===0" type="danger">否</el-tag>
          <el-tag v-else="scope.row.intention===1">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="room" header-align="center" align="center" label="住宿要求">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.room===0" type="danger">否</el-tag>
          <el-tag v-else="scope.row.room===1">是</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="transfer" header-align="center" align="center" label="接送要求">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.transfer===0" type="danger">否</el-tag>
          <el-tag v-else="scope.row.transfer===1">是</el-tag>
        </template>
      </el-table-column>-->
      <el-table-column prop="intention" header-align="center" align="center" label="参会意向">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.intention===1"
            type="success"
            @click="changeStatus(scope.row.id, 'intention', 0)"
          >是</el-button>
          <el-button v-else @click="changeStatus(scope.row.id, 'intention', 1)">否</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="room" header-align="center" align="center" label="住宿要求">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.room===1"
            type="success"
            @click="changeStatus(scope.row.id, 'room', 0)"
          >是</el-button>
          <el-button v-else @click="changeStatus(scope.row.id, 'room', 1)">否</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="transfer" header-align="center" align="center" label="接送要求">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.transfer===1"
            type="success"
            @click="changeStatus(scope.row.id, 'transfer', 0)"
          >是</el-button>
          <el-button v-else @click="changeStatus(scope.row.id, 'transfer', 1) ">否</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="transfer" header-align="center" align="center" label="用户类型"></el-table-column>
      <el-table-column prop="servicer" header-align="center" align="center" label="负责人"></el-table-column>
      <el-table-column prop="note" header-align="center" align="center" label="备注"></el-table-column>
      <!-- <el-table-column
        prop="account"
        header-align="center"
        align="center"
        label="联系电话">
      </el-table-column>-->
      <!-- <el-table-column
        prop="emailLx"
        header-align="center"
        align="center"
        label="联系邮箱">
      </el-table-column>-->
      <!-- <el-table-column
        prop="regflag"
        header-align="center"
        align="center"
        label="注册情况 0, 1, 2, 现场注册"
      >
      </el-table-column>-->

      <!-- <el-table-column prop="createTime" header-align="center" align="center" label="创建时间"></el-table-column> -->
      <el-table-column prop="modifyTime" header-align="center" align="center" label="修改时间"></el-table-column>
      <!-- <el-table-column prop="creater" header-align="center" align="center" label="创建者"></el-table-column>
      <el-table-column prop="modifier" header-align="center" align="center" label="修改者"></el-table-column>-->
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
import AddOrUpdate from "./attenders-add-or-update";
export default {
  data() {
    return {
      dataForm: {
        key: ""
      },
      dataList: [],
      pageIndex: 0,
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
        url: this.$http.adornUrl("/admin/attenders/list"),
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
    handclick(id) {
      window.console.log(id);
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
          url: this.$http.adornUrl("/admin/attenders/delete"),
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
    changeStatus(id, column, value) {
      console.log(id);
      // console.log(obj)
      this.$http({
        url: this.$http.adornUrl("/admin/attenders/status"),
        method: "post",
        data: this.$http.adornData({
          id: id,
          column: column,
          value: value
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 500,
            onClose: () => {
              //  this.getDataList()
              var dList = this.dataList;
              dList.forEach(element => {
                if (element.id === id) {
                  element[column] = value;
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
