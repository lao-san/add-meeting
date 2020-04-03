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
      <el-form-item label="密码" prop="password" v-if="!dataForm.id">
        <el-input v-model="dataForm.password" placeholder="密码"></el-input>
      </el-form-item>
      <el-form-item label="头像" prop="titlePic">
        <el-input v-model="dataForm.titlePic" placeholder="头像"></el-input>
      </el-form-item>
      <el-row>
        <img :src="showimg" alt="" style="width:300px;">
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
      <el-form-item label="办公电话" prop="officephone">
        <el-input v-model="dataForm.officephone" placeholder="办公电话"></el-input>
      </el-form-item>
       <el-form-item label="其他联系方式" prop="otherphone">
        <el-input v-model="dataForm.otherphone" placeholder="其他联系方式"></el-input>
      </el-form-item>
      <!-- <el-form-item label="审核" prop="isCheck">
        <el-radio v-model="dataForm.isCheck" :label="0">否</el-radio>
        <el-radio v-model="dataForm.isCheck" :label="1">是</el-radio>
      </el-form-item> -->
       <el-form-item label="负责人" prop="servicerId">
        <!-- <el-input v-model="dataForm.email" placeholder="其他联系方式"></el-input> -->
        <template>
          <el-select
            v-model="dataForm.servicerId"
            filterable
            remote
            reserve-keyword
            placeholder="请输入工作人员姓名"
            @change="$forceUpdate()"
            :remote-method="selectEmployeeIdByname"
            :loading="loading">
            <el-option
              v-for="item in employeeList"
              :key="item.id"
              :label="item.truename"
              :value="item.id">
            </el-option>
          </el-select>
        </template>
      </el-form-item>
       <el-form-item label="备注" prop="note">
        <el-input  type="textarea" v-model="dataForm.note" :rows="3" placeholder="备注"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">保存</el-button>
    </span>
    <div v-if="dataForm.id">
      <h3>参会信息</h3>
      <span>参会次数: </span><span> {{this.meetingList.length}}</span>
      <div>
        <span>选择会议</span>
        <el-select v-model="meetingId" placeholder="请输入选择会议">
          <el-option
            v-for="item in meetingList"
            :key="item.meeting_id"
            :label="item.name_cn"
            :value="item.meeting_id">
          </el-option>
        </el-select>
        <span>查看</span>
      </div>
    </div>
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
        officephone: '',
        otherphone: '',
        servicerId: 0,
        note: ''
      },
      dataRule: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' }
        ],
        truename: [
          { required: true, message: '姓名不能为空', trigger: 'blur' }
        ],
        password: [{ validator: checkPwd, trigger: 'blur' }],
        phone: [{ validator: checkPhone, trigger: 'blur' }],
        email: [{ validator: checkEmail, trigger: 'blur' }]
      },
      upload_url: '',
      radio: '0',
      thumb: {'isthumb': 0},
      showimg: '',
      employeeList: [],
      loading: false,
      meetingList: [],
      meetingId: ''
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
              this.employeeList = [{'id': data.employee.id, 'truename': data.employee.truename}]
              this.meetingList = data.meetingList
              // console.log(data.member)
              this.dataForm.username = data.member.username
              this.dataForm.truename = data.member.truename
              this.dataForm.titlePic = data.member.titlePic
              this.dataForm.organization = data.member.organization
              this.dataForm.position = data.member.position
              this.dataForm.jobTitle = data.member.jobTitle
              this.dataForm.phone = data.member.phone
              this.dataForm.email = data.member.email
              this.dataForm.officephone = data.member.officephone
              this.dataForm.otherphone = data.member.otherphone
              this.dataForm.servicerId = data.member.servicerId
              this.dataForm.note = data.member.note
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
              'id': this.dataForm.id || undefined,
              'username': this.dataForm.username,
              'password': this.dataForm.password,
              'truename': this.dataForm.truename,
              'sex': this.dataForm.sex,
              'titlePic': this.dataForm.titlePic,
              'organization': this.dataForm.organization,
              'position': this.dataForm.position,
              'jobTitle': this.dataForm.jobTitle,
              'phone': this.dataForm.phone,
              'email': this.dataForm.email,
              'officephone': this.dataForm.officephone,
              'otherphone': this.dataForm.otherphone,
              'servicerId': this.dataForm.servicerId,
              'note': this.dataForm.note
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
    },
    // 通过姓名模糊查询参会人员id
    selectEmployeeIdByname (name) {
      this.loading = true
      this.$http({
        url: this.$http.adornUrl(`/admin/employee/listbyname`),
        method: 'get',
        params: this.$http.adornParams({
          'truename': name
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.loading = false
          this.employeeList = data.list
        }
      })
    }
  }
}
</script>
