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
      <el-form-item label="姓名" prop="state2">
        <el-autocomplete
          class="inline-input"
          v-model="dataForm.state2"
          :fetch-suggestions="querySearch"
          placeholder="请输入内容"
          :trigger-on-focus="false"
          @select="handleSelect"
          :style="!dataForm.id ? 'width:80%' :'width:100%' "
        ></el-autocomplete>
        <el-button
          @click="creatorUser"
          class="ll-button"
          type="primary"
          v-show="!dataForm.id"
        >添加新的会员</el-button>
      </el-form-item>

      <el-form-item label="所属机构" prop="organization">
        <el-input v-model="dataForm.organization" placeholder="所属机构(单位名称、公司名称)"></el-input>
      </el-form-item>
      <el-form-item label="职位">
        <el-input v-model="dataForm.position" placeholder="职位"></el-input>
      </el-form-item>
      <el-form-item label="职称">
        <el-input v-model="dataForm.jobTitle" placeholder="职称"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="phone">
        <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
      </el-form-item>
      <el-form-item label="邮箱">
        <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
      </el-form-item>
      <el-form-item label="办公电话">
        <el-input v-model="dataForm.officephone" placeholder="办公电话"></el-input>
      </el-form-item>

      <el-form-item label="参会意向" prop="intention">
        <!-- <el-input v-model="dataForm.intention" placeholder="参会意向"></el-input> -->
        <el-radio v-model="dataForm.intention" :label="1">是</el-radio>
        <el-radio v-model="dataForm.intention" :label="0">否</el-radio>
      </el-form-item>
      <el-form-item label="住宿要求" prop="room">
        <!-- <el-input v-model="dataForm.room" placeholder="住宿要求"></el-input> -->
        <el-radio v-model="dataForm.room" :label="1">是</el-radio>
        <el-radio v-model="dataForm.room" :label="0">否</el-radio>
      </el-form-item>
      <el-form-item label="接送要求" prop="transfer">
        <!-- <el-input v-model="dataForm.transfer" placeholder="接送要求"></el-input> -->
        <el-radio v-model="dataForm.transfer" :label="1">是</el-radio>
        <el-radio v-model="dataForm.transfer" :label="0">否</el-radio>
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
      <el-form-item label="用户类型" prop="typeId">
        <el-select v-model="dataForm.typeId" filterable placeholder="请选择">
          <el-option
            v-for="item in typesofattendersList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="备注">
        <el-input
          type="textarea"
          :autosize="{ minRows: 2, maxRows: 4}"
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
      loading: false,
      dataForm: {
        id: 0,
        meetingId: "",
        memberId: "",
        name: "",
        organization: "",
        position: "",
        jobTitle: "",
        phone: "",
        email: "",
        officephone: "",
        typeId: "",
        intention: "",
        room: "",
        transfer: "",
        servicer: "",
        note: "",
        account: "",
        emailLx: "",
        regflag: "",
        state2: ""
      },
      dataRule: {
        meetingId: [
          { required: true, message: "会议id不能为空", trigger: "blur" }
        ],

        // state2: [{ required: true, message: "没有会员信息", trigger: "blur" }],
        organization: [
          {
            required: true,
            message: "所属机构(单位名称、公司名称)不能为空",
            trigger: "blur"
          }
        ],
        phone: [{ required: true, message: "电话不能为空", trigger: "blur" }],
        email: [{ required: true, message: "邮箱不能为空", trigger: "blur" }],

        intention: [
          { required: true, message: "参会意向不能为空", trigger: "blur" }
        ],
        room: [
          { required: true, message: "住宿要求不能为空", trigger: "blur" }
        ],
        transfer: [
          { required: true, message: "接送要求不能为空", trigger: "blur" }
        ],
        servicer: [
          {
            required: true,
            message: "负责人（员工id）不能为空",
            trigger: "blur"
          }
        ]
      },
      employeeList: [],
      typesofattendersList: [],
      restaurants: []
    };
  },
  created() {
    this.getEmployee();
    this.getTypesofattenders();
    this.getMember();
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.dataForm.memberId = 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/attenders/info/${this.dataForm.id}`
            ),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            if (data && data.code === 0) {
              this.dataForm.meetingId = data.attenders.meetingId;
              this.dataForm.memberId = data.attenders.memberId;
              this.dataForm.name = data.attenders.name;
              this.dataForm.organization = data.attenders.organization;
              this.dataForm.position = data.attenders.position;
              this.dataForm.jobTitle = data.attenders.jobTitle;
              this.dataForm.phone = data.attenders.phone;
              this.dataForm.email = data.attenders.email;
              this.dataForm.officephone = data.attenders.officephone;
              this.dataForm.typeId = data.attenders.typeId;
              this.dataForm.intention = data.attenders.intention;
              this.dataForm.room = data.attenders.room;
              this.dataForm.transfer = data.attenders.transfer;
              this.dataForm.servicer = data.attenders.servicer;
              this.dataForm.note = data.attenders.note;
              this.dataForm.state2 = data.attenders.name;
            }
          });
        } else {
          this.dataForm.meetingId = "";
          this.dataForm.memberId = "";
          this.dataForm.name = "";
          this.dataForm.organization = "";
          this.dataForm.position = "";
          this.dataForm.jobTitle = "";
          this.dataForm.phone = "";
          this.dataForm.email = "";
          this.dataForm.officephone = "";
          this.dataForm.typeId = "";
          this.dataForm.intention = "";
          this.dataForm.room = "";
          this.dataForm.transfer = "";
          this.dataForm.servicer = "";
          this.dataForm.note = "";
          this.dataForm.state2 = "";
        }
      });
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs["dataForm"].validate(valid => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(
              `/admin/attenders/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              meetingId: this.$route.params.id,
              memberId: this.dataForm.memberId,
              name: this.dataForm.name,
              organization: this.dataForm.organization,
              position: this.dataForm.position,
              jobTitle: this.dataForm.jobTitle,
              phone: this.dataForm.phone,
              email: this.dataForm.email,
              officephone: this.dataForm.officephone,
              typeId: this.dataForm.typeId,
              intention: this.dataForm.intention,
              room: this.dataForm.room,
              transfer: this.dataForm.transfer,
              servicer: this.dataForm.servicer,
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
    //获取员工信息
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
    },
    //获取用户类型信息
    getTypesofattenders() {
      this.$http({
        url: this.$http.adornUrl(
          `/admin/typesofattenders/listbymid/${this.$route.params.id}`
        ),
        method: "get"
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.typesofattendersList = data.list;
        }
      });
    },
    createFilter(queryString) {
      return restaurant => {
        return (
          restaurant.value.toLowerCase().indexOf(queryString.toLowerCase()) ===
          0
        );
      };
    },
    querySearch(queryString, cb) {
      var restaurants = this.restaurants;
      var results = queryString
        ? restaurants.filter(this.createFilter(queryString))
        : restaurants;
      if (results.length === 0) {
        this.$message.error("没有会员信息,请添加会员信息");
      }
      cb(results);
    },
    //获取会员信息
    getMember() {
      this.$http({
        url: this.$http.adornUrl("/admin/member/listall"),
        method: "get"
      }).then(({ data }) => {
        if (data.code === 0 && data) {
          for (let i = 0; i < data.list.length; i++) {
            data.list[i].value =
              data.list[i].truename + "," + data.list[i].organization;
          }
          this.restaurants = data.list;
        }
      });
    },
    handleSelect(item) {
      if (item) {
        this.dataForm.jiobTitle = item.jiobTitle;
        this.dataForm.organization = item.organization;
        this.dataForm.phone = item.phone;
        this.dataForm.position = item.position;
        this.dataForm.memberId = item.id;
        this.dataForm.name = item.truename;
        this.dataForm.state2 = item.truename;
      }
    },
    creatorUser() {
      this.dataForm.name = this.dataForm.state2;
      if (this.dataForm.name) {
        this.$message({
          message: "添加成功",
          type: "success",
          duration: 1000
        });
      }
    }
  }
};
</script>
<style lang="scss" scoped>
.ll-button {
  position: absolute;
  top: 0;
  right: 0;
}
</style>
