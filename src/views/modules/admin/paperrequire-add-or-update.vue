<template>
  <el-dialog title="征文要求设置" :close-on-click-modal="false" :visible.sync="visible">
    <el-form
      :model="dataForm"
      :rules="dataRule"
      ref="dataForm"
      label-width="80px"
      v-loading="loading"
    >
      <el-form-item label="截止时间" prop="deadline">
        <el-date-picker
          size="small"
          v-model="dataForm.deadline"
          type="datetime"
          placeholder="开始时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="要求" prop="content">
        <el-input
          type="textarea"
          :rows="2"
          placeholder="请输入内容"
          v-model="dataForm.content"
          :autosize="{ minRows: 10, maxRows: 30}"
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
        id: 0,
        meetingId: 0,
        deadline: "",
        content: ""
      },
      dataRule: {
        deadline: [
          { required: true, message: "截止时间不能为空", trigger: "blur" }
        ],
        content: [{ required: true, message: "要求不能为空", trigger: "blur" }]
      },
      loading: true
    };
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (id) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/paperrequire/infobymid/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            this.loading = false;
            if (data.code === 0 && data.paperRequire !== null) {
              this.dataForm.id = data.paperRequire.id;
              this.dataForm.meetingId = data.paperRequire.meetingId;
              this.dataForm.deadline = data.paperRequire.deadline;
              this.dataForm.content = data.paperRequire.content;
            } else {
              this.dataForm.id = 0;
              this.dataForm.meetingId = id;
              this.dataForm.deadline = "";
              this.dataForm.content = "";
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
              `/admin/paperrequire/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              deadline: this.dataForm.deadline,
              meetingId: this.dataForm.meetingId,
              content: this.dataForm.content
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.$message({
                message: "操作成功",
                type: "success",
                duration: 1500,
                onClose: () => {
                  this.visible = false;
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
