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
      <el-form-item label="投稿人姓名" prop="attendersId">
        <!-- <el-input v-model="dataForm.attendersId" placeholder="参会人员id"></el-input> -->
        <el-select v-model="dataForm.attendersId" filterable placeholder="请选择">
          <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="论文题目" prop="title">
        <el-input v-model="dataForm.title" placeholder="论文题目"></el-input>
      </el-form-item>
      <el-form-item label="摘要" prop="summary">
        <el-input v-model="dataForm.summary" placeholder="摘要" type="textarea"></el-input>
      </el-form-item>
      <el-form-item label="上传投稿">
        <el-upload
          class="upload-demo"
          ref="upload"
          :action="upload_url"
          :on-success="successHandle"
          :auto-upload="false"
          name="upload_file"
        >
          <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
          <el-button
            style="margin-left: 10px;"
            size="small"
            type="success"
            @click="submitUpload"
          >上传到服务器</el-button>
          <div slot="tip" class="el-upload__tip" style="color:red">只能上传 pdf doc docx 文件</div>
        </el-upload>
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
        meetingId: "",
        attendersId: "",
        title: "",
        summary: "",
        paperurl: ""
      },
      dataRule: {
        attendersId: [
          { required: true, message: "参会人员id不能为空", trigger: "blur" }
        ],
        summary: [{ required: true, message: "摘要不能为空", trigger: "blur" }]
      },
      options: [],
      upload_url: "",
      successNum: 0
    };
  },
  created() {
    this.getPerson();
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/paper/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.meetingId = this.$route.params.id;
              this.dataForm.attendersId = data.paper.attendersId;
              this.dataForm.title = data.paper.title;
              this.dataForm.summary = data.paper.summary;
              this.dataForm.paperurl = data.paper.paperurl;
            }
          });
        }
      });
      this.upload_url = this.$http.adornUrl(
        `/sys/filemanager/uploadfile?token=${this.$cookie.get("token")}`
      );
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/paper/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              meetingId: this.$route.params.id,
              attendersId: this.dataForm.attendersId,
              title: this.dataForm.title,
              summary: this.dataForm.summary
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
    getPerson() {
      this.$http({
        url: this.$http.adornUrl("/admin/attenders/selectbyname"),
        methods: "get",
        params: {
          name: "",
          meetingId: this.$route.params.id
        }
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.options = data.list;
        }
      });
    },
    paperUpload() {},
    submitUpload() {
      this.$refs.upload.submit();
    },
    successHandle(response) {
      window.console.log(response)
    }
  }
};
</script>
