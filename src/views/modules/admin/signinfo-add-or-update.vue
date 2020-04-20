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
      <el-form-item label="姓名" prop="truename">
        <el-input v-model="dataForm.truename" placeholder="姓名"></el-input>
      </el-form-item>
      <el-form-item label="所属机构" prop="organization">
        <el-input v-model="dataForm.organization" placeholder="所属机构"></el-input>
      </el-form-item>

      <!-- <el-form-item label="职称" prop="jobTitle">
        <el-input v-model="dataForm.jobTitle" placeholder="职称"></el-input>
      </el-form-item>-->
      <el-form-item label="电话" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="电话"></el-input>
      </el-form-item>
      <el-form-item label="缴费金额" prop="money">
        <el-input v-model="dataForm.money" placeholder="缴费金额(单位元)"></el-input>
      </el-form-item>
      <el-form-item label="是否缴费" prop="isPay">
        <el-radio v-model="dataForm.isPay" :label="1">是</el-radio>
        <el-radio v-model="dataForm.isPay" :label="0">否</el-radio>
      </el-form-item>
      <el-form-item label="胸卡打印" prop="badge">
        <!-- <el-input v-model="dataForm.badge" placeholder="是否完成胸卡打印"></el-input> -->
        <el-radio v-model="dataForm.badge" :label="1">是</el-radio>
        <el-radio v-model="dataForm.badge" :label="0">否</el-radio>
      </el-form-item>
      <el-form-item label="负责人" prop="servicer">
        <el-select v-model="dataForm.servicer" filterable placeholder="请选择">
          <el-option
            v-for="item in employeeList"
            :key="item.id"
            :label="item.truename"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注" prop="note">
        <el-input
          type="textarea"
          :rows="2"
          :autosize="{ minRows: 15, maxRows:15}"
          placeholder="请输入内容"
          v-model="dataForm.note"
        ></el-input>
      </el-form-item>
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
        meetingId: this.$route.params.id,
        truename: "",
        organization: "",
        phone: "",
        servicer: "",
        money: 0,
        isPay: '',
        badge: '',
        note: ""
      },
      dataRule: {
        truename: [
          { required: true, message: "姓名不能为空", trigger: "blur" }
        ],
        organization: [
          {
            required: true,
            message: "所属机构(单位名称、公司名称不能为空",
            trigger: "blur"
          }
        ],
        position: [
          { required: true, message: "职位不能为空", trigger: "blur" }
        ],
        phone: [{ required: true, message: "电话不能为空", trigger: "blur" }],
        typeId: [
          { required: true, message: "参会人员类型不能为空", trigger: "blur" }
        ],
        servicer: [
          {
            required: true,
            message: "负责人不能为空",
            trigger: "blur"
          }
        ],
        isPay: [
          { required: true, message: "是否缴费不能为空", trigger: "blur" }
        ],
        badge: [
          {
            required: true,
            message: "是否完成胸卡打印不能为空",
            trigger: "blur"
          }
        ],
        money: [
          { required: true, message: "缴费金额不能为空", trigger: "blur" }
        ]
      },
      employeeList: []
    };
  },
  created() {
    this.getEmployee();
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
              `/admin/signinfo/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            window.console.log(data);
            if (data && data.code === 0) {
              this.dataForm.meetingId = data.signInfo.meetingId;
              this.dataForm.truename = data.signInfo.truename;
              this.dataForm.organization = data.signInfo.organization;
              this.dataForm.phone = data.signInfo.phone;
              this.dataForm.servicer = data.signInfo.servicer;
              this.dataForm.isPay = data.signInfo.isPay;
              this.dataForm.badge = data.signInfo.badge;
              this.dataForm.note = data.signInfo.note;
              this.dataForm.money = data.signInfo.money;
            }
          });
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/signinfo/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              meetingId: this.dataForm.meetingId,
              truename: this.dataForm.truename,
              organization: this.dataForm.organization,
              position: this.dataForm.position,
              phone: this.dataForm.phone,
              typeId: this.dataForm.typeId,
              servicer: this.dataForm.servicer,
              servername: this.dataForm.servername,
              isPay: this.dataForm.isPay,
              badge: this.dataForm.badge,
              note: this.dataForm.note
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
    },

    //获取负责人
    getEmployee() {
      this.loading = true;
      this.$http({
        url: this.$http.adornUrl(`/admin/employee/findAllByTrueName`),
        method: "get",
        params: this.$http.adornParams({
          truename: ""
        })
      }).then(({ data }) => {
        if (data) {
          this.loading = false;
          this.employeeList = data;
        }
      });
    }
  }
};
</script>
