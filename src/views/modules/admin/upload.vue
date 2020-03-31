<template>
  <el-dialog
    title="上传文件"
    :close-on-click-modal="false"
    @close="closeHandle"
    :visible.sync="visible">
    <el-upload
      :action="url"
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
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        url: '',
        radio: '0',
        thumb: {'isthumb': 0}
      }
    },
    methods: {
      init (id) {
        this.url = this.$http.adornUrl(`/sys/filemanager/uploadimg`)
        this.visible = true
      },
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
          this.visible = false
        } else {
          this.$message.error(response.msg)
        }
      }
      // // 弹窗关闭时
      // closeHandle () {
      // }
    }
  }
</script>
