<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="会议id" prop="meetingId">
      <el-input v-model="dataForm.meetingId" placeholder="会议id"></el-input>
    </el-form-item>
    <el-form-item label="参会人员id" prop="attendersId">
       <template>
        <el-select
          v-model="dataForm.attendersId"
          filterable
          remote
          reserve-keyword
          placeholder="请输入姓名"
          :remote-method="selectAttendersIdByname"
          :loading="loading">
          <el-option
            v-for="item in options"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </template>
      <!-- <el-input v-model="dataForm.attendersId" placeholder="演讲人id"></el-input> -->
    </el-form-item>
    <el-form-item label="题目" prop="topic">
      <el-input v-model="dataForm.topic" placeholder="题目"></el-input>
    </el-form-item>
    <el-form-item label="所属会场(主会场/分会场)" prop="branchId">
      <el-input v-model="dataForm.branchId" placeholder="所属会场(主会场/分会场)"></el-input>
    </el-form-item>
    <el-form-item label="演讲时间" prop="speechTime">
      <el-input v-model="dataForm.speechTime" placeholder="演讲时间"></el-input>
    </el-form-item>
    <el-form-item label="摘要" prop="summary">
      <el-input v-model="dataForm.summary" placeholder="摘要"></el-input>
    </el-form-item>
    <el-form-item label="創建時間" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="創建時間"></el-input>
    </el-form-item>
    <el-form-item label="是否被删除 状态  0：正常   1：删除" prop="isDel">
      <el-input v-model="dataForm.isDel" placeholder="是否被删除 状态  0：正常   1：删除"></el-input>
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
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          meetingId: '',
          attendersId: '',
          topic: '',
          branchId: '',
          speechTime: '',
          summary: '',
          createTime: '',
          isDel: ''
        },
        dataRule: {
          meetingId: [
            { required: true, message: '会议id不能为空', trigger: 'blur' }
          ],
          // attendersId: [
          //   { required: true, message: '参会人员id不能为空', trigger: 'blur' }
          // ],
          topic: [
            { required: true, message: '题目不能为空', trigger: 'blur' }
          ],
          branchId: [
            { required: true, message: '所属会场(主会场/分会场)不能为空', trigger: 'blur' }
          ],
          speechTime: [
            { required: true, message: '演讲时间不能为空', trigger: 'blur' }
          ],
          summary: [
            { required: true, message: '摘要不能为空', trigger: 'blur' }
          ]
        },
        options: [],
        loading: false,
        meetingId: 0
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/admin/lecture/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.meetingId = data.lecture.meetingId
                this.dataForm.attendersId = data.lecture.attendersId
                this.dataForm.topic = data.lecture.topic
                this.dataForm.branchId = data.lecture.branchId
                this.dataForm.speechTime = data.lecture.speechTime
                this.dataForm.summary = data.lecture.summary
                this.dataForm.createTime = data.lecture.createTime
                this.dataForm.isDel = data.lecture.isDel
              }
            })
          }
        })
        this.meetingId = this.$route.params.id
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/admin/lecture/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'meetingId': this.dataForm.meetingId,
                'attendersId': this.dataForm.attendersId,
                'topic': this.dataForm.topic,
                'branchId': this.dataForm.branchId,
                'speechTime': this.dataForm.speechTime,
                'summary': this.dataForm.summary,
                'createTime': this.dataForm.createTime,
                'isDel': this.dataForm.isDel
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 通过姓名模糊查询参会人员id
      selectAttendersIdByname (name) {
        this.loading = true
        this.$http({
          url: this.$http.adornUrl(`/admin/attenders/selectbyname`),
          method: 'get',
          params: this.$http.adornParams({
            'name': name,
            'meetingId': this.meetingId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.loading = false
            this.options = data.list
          }
        })
      }
    }
  }
</script>
