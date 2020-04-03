<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="投稿人" prop="attendersId">
      <!-- <el-input v-model="dataForm.attendersId" placeholder="参会人员id"></el-input> -->
      <template>
        <el-select
          v-model="dataForm.attendersId"
          filterable
          remote
          reserve-keyword
          placeholder="请输入姓名"
          @change="$forceUpdate()"
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
    </el-form-item>
    <el-form-item label="论文题目" prop="title">
      <el-input v-model="dataForm.title" placeholder="论文题目"></el-input>
    </el-form-item>
    <el-form-item label="摘要" prop="summary">
      <el-input v-model="dataForm.summary" placeholder="摘要"></el-input>
    </el-form-item>
    <el-form-item label="论文存放地址" prop="paperurl">
      <el-input v-model="dataForm.paperurl" placeholder="论文存放地址"></el-input>
    </el-form-item>
      <el-upload
        :action="upload_url"
        ref="upload"
        :before-upload="beforeUploadHandle"
        :on-success="successHandle"
        name="upload_file"
        :auto-upload="false"
        :data="oldname"
        style="text-align: center;">
        <i class="el-icon-upload"></i>
        <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
        <div class="el-upload__tip" slot="tip">只支持pdf、doc、docx格式的文件</div>
        <span>是否使用原文件名</span>
        <el-radio v-model="radio" label="1">是</el-radio>
        <el-radio v-model="radio" label="0" >否</el-radio>
        <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
      </el-upload>
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
          attendersId: '',
          title: '',
          summary: '',
          paperurl: ''
        },
        dataRule: {
          attendersId: [
            { required: true, message: '参会人员id不能为空', trigger: 'blur' }
          ],
          title: [
            { required: true, message: '论文题目不能为空', trigger: 'blur' }
          ],
          summary: [
            { required: true, message: '摘要不能为空', trigger: 'blur' }
          ]
        },
        upload_url: '',
        radio: '0',
        oldname: {'useoldname': 0},
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
              url: this.$http.adornUrl(`/admin/paper/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.options = [{'id': data.attenders.id, 'name': data.attenders.name}]
                this.dataForm.attendersId = data.paper.attendersId
                this.dataForm.title = data.paper.title
                this.dataForm.summary = data.paper.summary
                this.dataForm.paperurl = data.paper.paperurl
              }
            })
          }
        })
        // 上传路径
        this.upload_url = this.$http.adornUrl(`/sys/filemanager/uploadimg?token=${this.$cookie.get('token')}`)
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/admin/paper/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'attendersId': this.dataForm.attendersId,
                'title': this.dataForm.title,
                'summary': this.dataForm.summary,
                'paperurl': this.dataForm.paperurl
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
      // 上传图像
      submitUpload () {
        this.$refs.upload.submit()
      },
      // 上传之前
      beforeUploadHandle (file) {
        this.oldname.useoldname = this.radio
        if (file.type !== 'application/pdf' && file.type !== 'application/msword') {
          this.$message.error('只支持jpg、png、gif格式的图片！')
          return false
        }
      },
      // 上传成功
      successHandle (response) {
        console.log(response)
        if (response && response.code === 0) {
          this.dataForm.paperurl = response.filename
        } else {
          this.$message.error(response.msg)
        }
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
