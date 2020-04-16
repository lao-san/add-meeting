<template>
  <div class="typesoffeeContent" v-loading="loading">
    <el-form>
      <el-form-item label="注册费用信息"></el-form-item>
      <el-form-item>
        <el-input style="width:400px" placeholder="请输入级别类型" v-model="dataForm.name"></el-input>
        <el-input style="width:400px" placeholder="请输入具体金额（单位：元）" v-model="dataForm.money"></el-input>
        <el-button style="width:180px" type="primary" @click="typesoffeeSave">添加</el-button>
      </el-form-item>
    </el-form>
    <el-table border :data="list">
      <el-table-column prop="name" label="级别类型" header-align="center" align="center"></el-table-column>
      <el-table-column prop="money" label="金额（单位：元）" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" header-align="center" align="center">
        <template slot-scope="scope">
          <el-button size="mini" type="danger" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-form
      :model="moneyaccountList"
      ref="moneyaccountList"
      style="margin-top:30px"
      class="form"
      label-position="right"
      label-width="130px"
    >
      <el-form-item label="对公收款账户信息"></el-form-item>
      <el-form-item label="账  号">
        <div v-if="show">{{moneyaccount.name}}</div>
        <el-input v-else="show" style="width:300px" size="small" v-model="moneyaccountList.name"></el-input>
      </el-form-item>
      <el-form-item label="账户名称">
        <div v-if="show">{{moneyaccount.bank}}</div>
        <el-input v-else="show" style="width:300px" size="small" v-model="moneyaccountList.bank"></el-input>
      </el-form-item>
      <el-form-item label="开户银行">
        <div v-if="show">{{moneyaccount.account}}</div>
        <el-input v-else="show" style="width:300px" size="small" v-model="moneyaccountList.account"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button
          class="button"
          style="width:100px"
          type="primary"
          v-if="show"
          @click="show=false"
        >修 改</el-button>
        <el-button
          class="button"
          style="width:100px"
          type="primary"
          v-else="show"
          @click=" pushMoneyaccount()"
        >保 存</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  name: "",
  data() {
    return {
      dataForm: {
        name: "",
        money: ""
      },
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      list: [], //注册费用信息
      moneyaccountList: {
        name: "",
        bank: "",
        account: ""
      },
      moneyaccount: {}, //注册类型返回数据
      show: true,
      loading: false
    };
  },
  components: {},
  computed: {},
  beforeMount() {},
  mounted() {
    this.getDataList();
    this.getMoneyaccount();
  },
  methods: {
    //提交
    typesoffeeSave() {
      this.$http({
        url: this.$http.adornUrl("/admin/typesoffee/save"),
        method: "post",
        data: {
          meetingId: this.$route.params.id,
          name: this.dataForm.name,
          money: this.dataForm.money
        }
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: "添加成功",
            type: "success",
            duration: 1000,
            onClose: () => {}
          });
          this.getDataList();
          this.dataForm.name = "";
          this.dataForm.money = "";
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    // 获取数据
    getDataList() {
      this.$http({
        url: this.$http.adornUrl("/admin/typesoffee/list/"),
        method: "get",
        params: {
          page: this.pageIndex,
          limit: this.pageSize,
          meetingId: this.$route.params.id
        }
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.list = data.page.list;
        }
      });
    },
    //删除
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
          url: this.$http.adornUrl("/admin/typesoffee/delete"),
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
    //提交对公收款账户信息
    pushMoneyaccount() {
      this.loading = true;
      this.$http({
        url: this.$http.adornUrl("/admin/moneyaccount/addorupdate"),
        method: "post",
        data: {
          meetingId: this.$route.params.id,
          name: this.moneyaccountList.name,
          bank: this.moneyaccountList.bank,
          account: this.moneyaccountList.account
        }
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.show = true;
          this.$message({
            message: "操作成功",
            type: "success",
            duration: 1500,
            onClose: () => {
              this.getMoneyaccount();
            }
          });
        } else {
          this.$message.error(data.msg);
        }
      });
    },
    //获取公收款账户信息
    getMoneyaccount() {
      this.$http({
        url: this.$http.adornUrl(
          `/admin/moneyaccount/infobymid/${this.$route.params.id}`
        ),
        methods: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.moneyaccount = data.moneyAccount;
          this.loading = false;
        }
      });
    }
  },
  watch: {}
};
</script>
<style scoped lang='scss'>
.typesoffeeContent {
  width: 1000px;
  height: 700px;
  margin: 0 auto;
  // background-color: red;
  // border: 1px solid red;
  .form {
    position: relative;
    .button {
      position: absolute;
      bottom: 83px;
      right: 7px;
    }
  }
}
</style>
