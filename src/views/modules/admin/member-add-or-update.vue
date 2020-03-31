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
      label-width="80px"
    >
      <el-form-item label="姓名" prop="truename">
        <el-input v-model="dataForm.truename" placeholder="姓名"></el-input>
      </el-form-item>
      <el-form-item label="用户名" prop="username">
        <el-input v-model="dataForm.username" placeholder="用户名"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="头像" prop="titlePic">
        <el-input v-model="dataForm.titlePic" placeholder="头像"></el-input>
      </el-form-item>
      <el-row>
        <img :src="showimg" alt="">
      </el-row>
      <el-upload
      :action="upload_url"
      ref="upload"
      :before-upload="beforeUploadHandle"
      :on-success="successHandle"
      name="upload_file"
      :auto-upload="false"
      :data="thumb"
      style="text-align: center;">
      <i class="el-icon-upload"></i>
       <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
      <div class="el-upload__tip" slot="tip">只支持jpg、png、gif格式的图片！只能上传jpg/png文件，且不超过2M</div>
      <el-radio v-model="radio" label="1">是</el-radio>
      <el-radio v-model="radio" label="0" >否</el-radio>
      <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
    </el-upload>
      <el-form-item label="所属机构" prop="organization">
        <el-input v-model="dataForm.organization" placeholder="所属机构"></el-input>
      </el-form-item>
      <el-form-item label="职位" prop="position">
        <el-input v-model="dataForm.position" placeholder="职位"></el-input>
      </el-form-item>
      <el-form-item label="职称" prop="jobTitle">
        <el-input v-model="dataForm.jobTitle" placeholder="职称"></el-input>
      </el-form-item>
      <el-form-item label="电话" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="电话"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item label="审核" prop="isCheck">
        <el-radio v-model="dataForm.isCheck" :label="0">否</el-radio>
        <el-radio v-model="dataForm.isCheck" :label="1">是</el-radio>
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
    var checkPwd = (rule, value, callback) => {
      if (!value) {
        if (this.dataForm.id === 0) {
          return callback(new Error('密码不可为空'))
        } else {
          return callback()
        }
      } else {
        if (!value.match(/^.*(?=.{6,})/)) {
          return callback(new Error('密码需6位或以上'))
        }
        return callback()
      }
    }

    var checkEmail = (rule, value, callback) => {
      if (!value) {
        return callback()
      } else {
        if (
          !value.match(/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/)
        ) {
          return callback(new Error('邮箱格式错误'))
        }
        return callback()
      }
    }

    var checkPhone = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('电话不能为空'))
      } else {
        if (!value.match(/^1(3|4|5|6|7|8)\d{9}$/)) {
          return callback(new Error('电话格式错误'))
        } else {
          return callback()
        }
      }
    }
    return {
      add: false,
      visible: false,
      dataForm: {
        id: 0,
        username: '',
        password: '',
        truename: '',
        titlePic: '',
        organization: '',
        position: '',
        jobTitle: '',
        phone: '',
        email: '',
        createTime: '',
        isCheck: 0
      },
      dataRule: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' }
        ],
        truename: [
          { required: true, message: '姓名不能为空', trigger: 'blur' }
        ],
        // password: [
        //   { required: true, message: '密码不能为空', trigger: 'blur' }
        // ],
        // phone: [
        //   { required: true, message: '电话不能为空', trigger: 'blur' }
        // ]
        password: [{ validator: checkPwd, trigger: 'blur' }],
        phone: [{ validator: checkPhone, trigger: 'blur' }],
        email: [{ validator: checkEmail, trigger: 'blur' }]
      },
      upload_url: '',
      radio: '0',
      thumb: {'isthumb': 0},
      showimg: ''
    }
  },
  methods: {
    init (id) {
      this.dataForm.id = id || 0
      if (this.dataForm.id === 0) {
        this.add = true
      }
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/member/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              console.log(data.member)
              this.dataForm.username = data.member.username
              this.dataForm.truename = data.member.truename
              this.dataForm.titlePic = data.member.titlePic
              this.dataForm.organization = data.member.organization
              this.dataForm.position = data.member.position
              this.dataForm.jobTitle = data.member.jobTitle
              this.dataForm.phone = data.member.phone
              this.dataForm.email = data.member.email
              this.dataForm.createTime = data.member.createTime
              this.dataForm.modifyTime = data.member.modifyTime
              this.dataForm.isCheck = data.member.isCheck
              this.showimg = 'http://121.42.53.174:9008/static' + data.member.titlePic
            }
          })
        }
      })
      // 上传路径
      this.upload_url = this.$http.adornUrl(`/sys/filemanager/uploadimg?token=${this.$cookie.get('token')}`)
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/member/${!this.dataForm.id ? 'save' : 'update'}`
            ),
            method: 'post',
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              username: this.dataForm.username,
              password: this.dataForm.password,
              truename: this.dataForm.truename,
              sex: this.dataForm.sex,
              titlePic: this.dataForm.titlePic,
              organization: this.dataForm.organization,
              position: this.dataForm.position,
              jobTitle: this.dataForm.jobTitle,
              phone: this.dataForm.phone,
              email: this.dataForm.email,
              createTime: this.dataForm.createTime,
              modifyTime: this.dataForm.modifyTime,
              isCheck: this.dataForm.isCheck,
              isDel: this.dataForm.isDel
            })
          }).then(({ data }) => {
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
      this.thumb.isthumb = this.radio
      if (file.type !== 'image/jpg' && file.type !== 'image/jpeg' && file.type !== 'image/png' && file.type !== 'image/gif') {
        this.$message.error('只支持jpg、png、gif格式的图片！')
        return false
      }
    },
    // 上传成功
    successHandle (response) {
      console.log(response)
      if (response && response.code === 0) {
        if (response.hasOwnProperty('thumb')) {
          this.dataForm.titlePic = response.thumb
          this.showimg = 'http://121.42.53.174:9008/static' + response.thumb
        } else {
          this.showimg = 'http://121.42.53.174:9008/static' + response.picname
          this.dataForm.titlePic = response.picname
        }
      } else {
        this.$message.error(response.msg)
      }
    }
  }
}
</script>
