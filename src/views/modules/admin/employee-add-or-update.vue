<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible"
  >
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      @keyup.enter.native="dataFormSubmit()"
      label-width="100px"
    >
      <el-form-item label="账号" prop="username">
        <el-input v-model="dataForm.username" placeholder="账号"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="姓名" prop="truename">
        <el-input v-model="dataForm.truename" placeholder="姓名"></el-input>
      </el-form-item>
      <el-form-item label="单位" prop="companyId">
        <!-- <el-input v-model="dataForm.companyId" placeholder="单位id"></el-input> -->
        <el-select v-model="dataForm.companyId" placeholder="请选择">
          <el-option
            v-for="item in dataList"
            :key="item.id"
            :label="item.companyname"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <!-- <el-form-item label="角色id（控制权限）" prop="roleId">
        <el-input v-model="dataForm.roleId" placeholder="角色id（控制权限）"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="部门id" prop="departmentId">
        <el-input v-model="dataForm.departmentId" placeholder="部门id"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="职位id" prop="positionId">
        <el-input v-model="dataForm.positionId" placeholder="职位id"></el-input>
      </el-form-item>-->
      <el-form-item label="联系电话" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="联系电话"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <!-- <el-form-item label="创建时间" prop="createTime">
        <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
      </el-form-item>
      <el-form-item label="修改时间" prop="modifyTime">
        <el-input v-model="dataForm.modifyTime" placeholder="修改时间"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="是否被删除 状态  0：正常   1：删除" prop="isDel">
        <el-input v-model="dataForm.isDel" placeholder="是否被删除 状态  0：正常   1：删除"></el-input>
      </el-form-item>-->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      visible: false,
      dataForm: {
        username: "",
        password: "",
        truename: "",
        companyId: "",
        // roleId: "",
        // departmentId: "",
        // positionId: "",
        phone: "",
        email: ""
      },
      dataRule: {
        username: [
          { required: true, message: "账号不能为空", trigger: "blur" }
        ],
        password: [
          { required: true, message: "密码不能为空", trigger: "blur" }
        ],
        truename: [
          { required: true, message: "姓名不能为空", trigger: "blur" }
        ],
        // companyId: [
        //   { required: true, message: "单位id不能为空", trigger: "blur" }
        // ],
        // roleId: [
        //   {
        //     required: true,
        //     message: "角色id（控制权限）不能为空",
        //     trigger: "blur"
        //   }
        // ],
        // departmentId: [
        //   { required: true, message: "部门id不能为空", trigger: "blur" }
        // ],
        // positionId: [
        //   { required: true, message: "职位id不能为空", trigger: "blur" }
        // ],
        phone: [
          { required: true, message: "联系电话不能为空", trigger: "blur" }
        ],
        email: [{ required: true, message: "邮箱不能为空", trigger: "blur" }],
        createTime: [
          { required: true, message: "创建时间不能为空", trigger: "blur" }
        ]
      },
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataList: []
    };
  },
  created() {
    this.getCompany();
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/employee/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.username = data.employee.username;
              this.dataForm.password = data.employee.password;
              this.dataForm.truename = data.employee.truename;
              this.dataForm.companyId = data.employee.companyId;
              this.dataForm.roleId = data.employee.roleId;
              this.dataForm.departmentId = data.employee.departmentId;
              this.dataForm.positionId = data.employee.positionId;
              this.dataForm.phone = data.employee.phone;
              this.dataForm.email = data.employee.email;
              this.dataForm.createTime = data.employee.createTime;
              this.dataForm.modifyTime = data.employee.modifyTime;
            }
          });
        }
      });
    },
    // /admin/company/list
    // getCompany() {
    //   this.$http({
    //     url: this.$http.adornUrl("/admin/company/list"),
    //     methods: "get"
    //   });
    // },
    getCompany() {
      this.$http({
        url: this.$http.adornUrl("/admin/company/list"),
        method: "get",
        params: this.$http.adornParams({
          page: this.pageIndex,
          limit: this.pageSize
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

    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/employee/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              username: this.dataForm.username,
              password: this.dataForm.password,
              truename: this.dataForm.truename,
              companyId: this.dataForm.companyId,
              phone: this.dataForm.phone,
              email: this.dataForm.email
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
                  this.$emit("refreshDataList");
                }
              });
            } else {
              this.$message.error(data.msg);
            }
          });
        }
      });
    }
  }
};
</script>
