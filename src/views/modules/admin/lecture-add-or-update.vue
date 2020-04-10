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
      <!-- <el-form-item label="会议id" prop="meetingId">
        <el-input v-model="dataForm.meetingId" placeholder="会议id"></el-input>
      </el-form-item>-->
      <el-form-item label="参会人员" prop="attendersId">
        <!-- <el-select v-model="dataForm.attendersId" filterable placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>-->
        <el-select
          v-model="dataForm.attendersId"
          filterable
          remote
          reserve-keyword
          placeholder="请输入关键词"
          :remote-method="remoteMethod"
          :loading="loading"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>

        <!-- <el-input v-model="dataForm.attendersId" placeholder="演讲人id"></el-input> -->
      </el-form-item>
      <el-form-item label="题目" prop="topic">
        <el-input v-model="dataForm.topic" placeholder="题目"></el-input>
      </el-form-item>
      <!-- <el-form-item label="所属会场(主会场/分会场)" prop="branchId">
        <el-input v-model="dataForm.branchId" placeholder="所属会场(主会场/分会场)"></el-input>
      </el-form-item>-->
      <el-form-item label="演讲时间" prop="speechTime">
        <el-date-picker
          v-model="dataForm.speechTime"
          type="datetime"
          placeholder="选择日期时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="摘要" prop="summary">
        <el-input v-model="dataForm.summary" placeholder="摘要" type="textarea"></el-input>
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
        topic: "",
        branchId: "0",
        speechTime: "",
        summary: "",
        name: ""
      },
      dataRule: {
        meetingId: [
          { required: true, message: "会议id不能为空", trigger: "blur" }
        ],
        attendersId: [
          { required: true, message: "参会人员id不能为空", trigger: "blur" }
        ],
        topic: [{ required: true, message: "题目不能为空", trigger: "blur" }],
        branchId: [
          {
            required: true,
            message: "所属会场(主会场/分会场)不能为空",
            trigger: "blur"
          }
        ],
        speechTime: [
          { required: true, message: "演讲时间不能为空", trigger: "blur" }
        ],
        summary: [{ required: true, message: "摘要不能为空", trigger: "blur" }],
        createTime: [
          { required: true, message: "創建時間不能为空", trigger: "blur" }
        ],
        isDel: [
          {
            required: true,
            message: "是否被删除 状态  0：正常   1：删除不能为空",
            trigger: "blur"
          }
        ]
      },
      options: [],
      loading: false,
      meetingId: 0,
      list: [],
      meetingList: []
    };
  },
  created() {
    this.getAttendersIdByname();
  },
  mounted() {},

  methods: {
    init(id) {
      window.console.log(id);
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/lecture/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.meetingId = data.lecture.meetingId;
              this.dataForm.attendersId = data.lecture.attendersId;
              this.dataForm.topic = data.lecture.topic;
              this.dataForm.branchId = data.lecture.branchId;
              this.dataForm.speechTime = data.lecture.speechTime;
              this.dataForm.summary = data.lecture.summary;
              !this.dataForm.id
                ? this.dataForm.attendersId
                : this.dataForm.attendersId = "";
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
              `/admin/lecture/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              meetingId: this.$route.params.id,
              attendersId: this.dataForm.attendersId,
              topic: this.dataForm.topic,
              branchId: this.dataForm.branchId,
              speechTime: this.dataForm.speechTime,
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
    // 通过姓名模糊查询参会人员id
    getAttendersIdByname(name = "") {
      this.loading = true;
      this.$http({
        url: this.$http.adornUrl(`/admin/attenders/selectbyname`),
        method: "get",
        params: this.$http.adornParams({
          name: name,
          meetingId: this.$route.params.id
        })
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.loading = false;
          this.meetingList = data.list;
          this.list = this.meetingList.map(item => {
            return { value: `${item.id}`, label: `${item.name}` };
          });
        }
      });
    },
    remoteMethod(query) {
      if (query !== "") {
        this.loading = true;
        setTimeout(() => {
          this.loading = false;
          this.options = this.list.filter(item => {
            return item.label.toLowerCase().indexOf(query.toLowerCase()) > -1;
          });
        }, 300);
      } else {
        this.options = [];
      }
    }
  }
};
</script>
