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
      label-width="136px"
    >
      <!-- <el-form-item label="举办单位id" prop="companyId">
        <el-input v-model="dataForm.companyId" placeholder="举办单位id"></el-input>
      </el-form-item>-->
      <el-form-item label="会议名称(中文)" prop="nameCn">
        <el-input v-model="dataForm.nameCn" placeholder="会议名称(中文)"></el-input>
      </el-form-item>
      <el-form-item label="会议名称(英文)" prop="nameEn">
        <el-input v-model="dataForm.nameEn" placeholder="会议名称(英文)"></el-input>
      </el-form-item>
      <el-form-item label="举办地点" prop="address">
        <el-input v-model="dataForm.address" placeholder="举办地点"></el-input>
      </el-form-item>
      <el-form-item label="开始时间" prop="startTime">
        <!-- <el-input v-model="dataForm.startTime" placeholder="开始时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.startTime"
          type="datetime"
          placeholder="开始时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <!-- <el-input v-model="dataForm.endTime" placeholder="结束时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.endTime"
          type="datetime"
          placeholder="结束时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="官方网址" prop="officeWebsite">
        <el-input v-model="dataForm.officeWebsite" placeholder="官方网址">
          <template slot="prepend">Http://</template>
        </el-input>
      </el-form-item>
      <el-form-item label="上传会议封面图片" prop="titlePicture">
        <!-- <el-input v-model="dataForm.titlePicture" placeholder="上传会议封面图片"></el-input> -->
        <el-upload
          :action="dialogImageUrl"
          list-type="picture-card"
          :on-remove="handleRemove"
          :on-success="successHandle"
          name="upload_file"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt />
        </el-dialog>
      </el-form-item>

      <el-form-item label="涉及学科" prop="subjects">
        <el-autocomplete
          v-model="dataForm.subjects"
          :fetch-suggestions="querySearchAsync"
          placeholder="每次添加一个学科"
          style="width:690px"
        ></el-autocomplete>
        <el-button @click="handSubject">添加</el-button>
      </el-form-item>
      <el-form-item v-if="this.subjects.length!==0">
        <el-tag v-for="(item) in subjects" style="margin-right:8px">{{item}}</el-tag>
      </el-form-item>
      <el-form-item label="涉及行业" prop="industries">
        <el-autocomplete
          v-model="dataForm.industries"
          :fetch-suggestions="queryIndustriesAsync"
          placeholder="每次添加一个行业"
          style="width:690px"
        ></el-autocomplete>
        <el-button @click="handIndustries">添加</el-button>
      </el-form-item>
      <el-form-item v-if="this.industry.length!==0">
        <el-tag v-for="(item) in industry" style="margin-right:8px">{{item}}</el-tag>
      </el-form-item>
      <el-form-item label="参会人员类型" prop="attendersType">
        <el-autocomplete
          v-model="dataForm.attendersType"
          :fetch-suggestions="queryAttendersTypeAsync"
          placeholder="每次添加一个类型"
          style="width:690px"
        ></el-autocomplete>
        <el-button @click="handAttendersType">添加</el-button>
      </el-form-item>
      <el-form-item v-if="this.typesofattender.length!==0">
        <el-tag v-for="(item) in typesofattender" style="margin-right:8px">{{item}}</el-tag>
      </el-form-item>
      <el-form-item label="联系人" prop="serviceEm">
        <el-select
          v-model="dataForm.contactNames"
          filterable
          placeholder="请选择"
          style="width:220px"
          @change="handEmploye"
        >
          <el-option
            v-for="item in employeeList"
            :key="item.id"
            :label="item.truename"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item v-if="this.employeePersonId.length!==0">
        <el-tag v-for="(item) in employeePerson" :key="item" style="margin-right:8px">{{item}}</el-tag>
      </el-form-item>

      <el-form-item label="会议负责人" prop="serviceEmp">
        <el-select v-model="dataForm.serviceIds" filterable placeholder="请选择" style="width:220px">
          <el-option
            v-for="item in employeeList"
            :key="item.id"
            :label="item.truename"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="线上报名开始时间" prop="onlineRegDeadline">
        <el-date-picker
          v-model="dataForm.onlineRegDeadline"
          type="datetime"
          placeholder="开始时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="现场报名截止时间" prop="onsiteRegDeadline">
        <el-date-picker
          v-model="dataForm.onsiteRegDeadline"
          type="datetime"
          placeholder="截止时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>

      <el-form-item label="会议介绍" prop="introduction">
        <el-input
          type="textarea"
          :autosize="{ minRows: 2, maxRows: 10}"
          placeholder="请输入内容"
          v-model="dataForm.introduction"
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
      url: "",
      imageUrl: "",
      visible: false,
      dialogVisible: false,

      dataForm: {
        companyId: 1,
        nameCn: "",
        nameEn: "",
        address: "",
        startTime: "",
        endTime: "",
        officeWebsite: "",
        titlePicture: "",
        subjects: "",
        industries: "",
        attendersType: "",
        serviceName: "",
        serviceIds: "",
        contactNames: "", // 联系人ids
        contactIds: "", // 联系人姓名
        onlineRegDeadline: "",
        onsiteRegDeadline: "",
        introduction: ""
      },

      dataRule: {
        companyId: [
          { required: true, message: "举办单位id不能为空", trigger: "blur" }
        ],
        nameCn: [
          { required: true, message: "会议名称(中文)不能为空", trigger: "blur" }
        ],
        nameEn: [
          { required: true, message: "会议名称(英文)不能为空", trigger: "blur" }
        ],
        address: [
          { required: true, message: "举办地点不能为空", trigger: "blur" }
        ],
        startTime: [
          { required: true, message: "开始时间不能为空", trigger: "blur" }
        ],
        endTime: [
          { required: true, message: "结束时间不能为空", trigger: "blur" }
        ],

        onlineRegDeadline: [
          {
            required: true,
            message: "线上报名截止时间不能为空",
            trigger: "blur"
          }
        ],
        onsiteRegDeadline: [
          {
            required: true,
            message: "现场报名截止时间不能为空",
            trigger: "blur"
          }
        ],
        subDeadline: [
          { required: true, message: "投稿截止时间不能为空", trigger: "blur" }
        ],
        subRequirement: [
          { required: true, message: "投稿范围及要求不能为空", trigger: "blur" }
        ],
        introduction: [
          { required: true, message: "会议介绍不能为空", trigger: "blur" }
        ],
        paperRequireId: [
          { required: true, message: "投稿要求id不能为空", trigger: "blur" }
        ],
        createTime: [
          { required: true, message: "创建时间不能为空", trigger: "blur" }
        ],
        modifyTime: [
          { required: true, message: "修改时间不能为空", trigger: "blur" }
        ],
        isCheck: [
          {
            required: true,
            message: "是否通过审核 0:未通过 1:通过不能为空",
            trigger: "blur"
          }
        ],
        isDel: [
          {
            required: true,
            message: "是否被删除 状态  0：正常   1：删除不能为空",
            trigger: "blur"
          }
        ]
      },
      dialogImageUrl: "",
      dialogVisible: false,
      titlePictureList: [],
      page: 0,
      limit: 10,
      key: "",
      restaurants: [], //学科表信息
      subjects: [], // 提交后台学科表
      industryRestaurants: [], //行业表信息
      industry: [], //提交后台行业表
      typesofattendersRestaurants: [], //参会人员表信息
      typesofattender: [], //参会人员提交后台
      employeeList: [], //员工表信息
      employeePersonId: [],
      employeePerson: []
    };
  },
  mounted() {
    this.getSubject();
    this.getIndustries();
    this.getAttendersType();
    this.getEmployeeList();
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/meeting/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.nameCn = data.meeting.nameCn;
              this.dataForm.nameEn = data.meeting.nameEn;
              this.dataForm.address = data.meeting.address;
              this.dataForm.startTime = data.meeting.startTime;
              this.dataForm.endTime = data.meeting.endTime;
              this.dataForm.officeWebsite = data.meeting.officeWebsite;
              this.dataForm.titlePicture = data.meeting.titlePicture;
              this.dataForm.subjects = data.meeting.subjects;
              this.dataForm.industries = data.meeting.industries;
              this.dataForm.attendersType = data.meeting.attendersType;
              this.dataForm.serviceEmp = data.meeting.serviceEmp;
              this.dataForm.onlineRegDeadline = data.meeting.onlineRegDeadline;
              this.dataForm.onsiteRegDeadline = data.meeting.onsiteRegDeadline;
              this.dataForm.introduction = data.meeting.introduction;
            }
          });
        }
      });
      this.dialogImageUrl = this.$http.adornUrl(
        `/sys/filemanager/uploadimg?token=${this.$cookie.get("token")}`
      );
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/meeting/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              nameCn: this.dataForm.nameCn,
              nameEn: this.dataForm.nameEn,
              address: this.dataForm.address,
              startTime: this.dataForm.startTime,
              endTime: this.dataForm.endTime,
              officeWebsite: this.dataForm.officeWebsite,
              titlePicture: this.titlePictureList.join(),
              subjects: this.subjects.join(),
              industries: this.industry.join(),
              attendersType: this.typesofattender.join(),
              serviceIds: this.dataForm.serviceIds,
              contactNames: this.employeePerson.join(),
              contactIds: this.employeePersonId.join(),
              onlineRegDeadline: this.dataForm.onlineRegDeadline,
              onsiteRegDeadline: this.dataForm.onsiteRegDeadline,
              introduction: this.dataForm.introduction
            })
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.subjects = [];
              this.titlePictureList = [];
              this.industry = [];
              this.typesofattender = [];
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
    // 上传会议图片
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    successHandle(response, file) {
      if (response && response.code === 0) {
        this.titlePictureList.push(response.picname);
      } else {
        this.$message.error(response.msg);
      }
    },
    //获取会议学科
    getSubject() {
      this.$http({
        url: this.$http.adornUrl("/admin/subject/list"),
        method: "get",
        params: {
          page: this.page,
          limit: this.limit
        }
      }).then(res => {
        if (res && res.data.code === 0) {
          this.restaurants = res.data.page.list;
          for (let i of this.restaurants) {
            i.value = i.name;
          }
        }
      });
    },
    //会议回显
    querySearchAsync(queryString, cb) {
      var restaurants = this.restaurants;
      var results = queryString
        ? restaurants.filter(this.createStateFilter(queryString))
        : restaurants;
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        cb(results);
      }, 1000 * Math.random());
    },
    createStateFilter(queryString) {
      return restaurant => {
        return (
          restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) ===
          0
        );
      };
    },
    //学科
    handSubject() {
      if (this.dataForm.subjects) {
        this.subjects.push(this.dataForm.subjects);
        this.dataForm.subjects = "";
      }
    },
    getIndustries() {
      this.$http({
        url: this.$http.adornUrl("/admin/industry/list"),
        method: "get",
        params: {
          page: this.page,
          limit: this.limit
        }
      }).then(res => {
        if (res && res.data.code === 0) {
          this.industryRestaurants = res.data.page.list;
          for (let i of this.industryRestaurants) {
            i.value = i.name;
          }
        }
      });
    },
    // 行业回显
    queryIndustriesAsync(queryString, cb) {
      let restaurants = this.industryRestaurants;
      let results = queryString
        ? restaurants.filter(this.createStateFilter(queryString))
        : restaurants;
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        cb(results);
      }, 1000 * Math.random());
    },

    //添加行业下tag渲染
    handIndustries() {
      if (this.dataForm.industries) {
        this.industry.push(this.dataForm.industries);
        this.dataForm.industries = "";
      }
    },
    //获取类型表
    getAttendersType() {
      this.$http({
        url: this.$http.adornUrl("/admin/typesofattenders/list"),
        method: "get",
        params: {
          page: this.page,
          limit: this.limit
        }
      }).then(res => {
        if (res && res.data.code === 0) {
          this.typesofattendersRestaurants = res.data.page.list;
          for (let i of this.typesofattendersRestaurants) {
            i.value = i.name;
          }
        }
      });
    },
    //添加参会人员类型
    queryAttendersTypeAsync(queryString, cb) {
      let restaurants = this.typesofattendersRestaurants;
      let results = queryString
        ? restaurants.filter(this.createStateFilter(queryString))
        : restaurants;
      clearTimeout(this.timeout);
      this.timeout = setTimeout(() => {
        cb(results);
      }, 1000 * Math.random());
    },
    handAttendersType() {
      if (this.dataForm.attendersType) {
        this.typesofattender.push(this.dataForm.attendersType);
        this.dataForm.attendersType = "";
      }
    },
    //获取员工列表模糊查询
    getEmployeeList() {
      this.$http({
        url: this.$http.adornUrl(`/admin/employee/findAllByTrueName`),
        method: "get",
        params: this.$http.adornParams({
          truename: ""
        })
      }).then(({ data }) => {
        if (data) {
          this.employeeList = data;
          for (let i of this.employeeList) {
            i.value = i.truename;
          }
        }
      });
    },
    //添加联系人
    handEmploye() {
      if (this.employeePersonId.length >= 3) {
        this.$message.error({
          message: "联系人最多添加三个",
          type: "danger",
          duration: 1500
        });
      } else {
        this.employeePersonId.push(this.dataForm.contactNames);
        for (let i = 0; i < this.employeeList.length; i++) {
          if (this.dataForm.contactNames === this.employeeList[i].id) {
            this.employeePerson.push(this.employeeList[i].truename);
          }
        }
      }
    }
  }
  
};
</script>
<style lang='scss'>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
