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
      label-width="150px"
    >
      <!-- <el-form-item label="订单号" prop="orderId">
        <el-input v-model="dataForm.orderId" placeholder="订单号"></el-input>
      </el-form-item>-->
      <el-form-item label="参会人员" prop="attenders">
        <!-- <el-input v-model="dataForm.attendersId" placeholder="参会人员id"></el-input> -->
        <el-select
          style="width:220px"
          v-model="dataForm.attenders"
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
      </el-form-item>
      <!-- <el-form-item label="费用类型id" prop="feeId">
        <el-input v-model="dataForm.feeId" placeholder="费用类型id"></el-input>
      </el-form-item>-->
      <el-form-item label="缴费时间" prop="payTime">
        <!-- <el-input v-model="dataForm.payTime" placeholder="缴费时间"></el-input> -->
        <el-date-picker
          v-model="dataForm.payTime"
          type="datetime"
          placeholder="选择日期时间"
          value-format="yyyy-MM-dd HH:mm:ss"
        ></el-date-picker>
      </el-form-item>
      <el-form-item label="费用类型" prop="feeId">
        <el-select v-model="dataForm.feeId" placeholder="请选择">
          <el-option
            v-for="item in option"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="缴费方式" prop="payType">
        <!-- <el-input v-model="dataForm.payType" placeholder="缴费方式"></el-input> -->
        <!-- <el-select v-model="dataForm.payType" placeholder="请选择">
          <el-option
            v-for="item in payTypeList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          ></el-option>
        </el-select>-->
        <el-radio v-model="dataForm.payType" :label="1">线上缴费</el-radio>
        <el-radio v-model="dataForm.payType" :label="2">现场缴费</el-radio>
      </el-form-item>

      <el-form-item label="缴费项目" prop="payOption">
        <el-input v-model="dataForm.payOption" placeholder="缴费项目"></el-input>
      </el-form-item>
      <el-form-item label="金额" prop="money">
        <el-input v-model="dataForm.money" placeholder="金额"></el-input>
      </el-form-item>
      <el-form-item label="发票抬头" prop="taxTitle">
        <el-input v-model="dataForm.taxTitle" placeholder="发票抬头"></el-input>
      </el-form-item>
      <el-form-item label="纳税人税号" prop="taxNumber">
        <el-input v-model="dataForm.taxNumber" placeholder="纳税人税号"></el-input>
      </el-form-item>
      <el-form-item label="公司注册地址及电话" prop="addrPhone">
        <el-input v-model="dataForm.addrPhone" placeholder="电话"></el-input>
      </el-form-item>
      <el-form-item label="开户行及账号" prop="bankAddrAccount">
        <el-input v-model="dataForm.bankAddrAccount" placeholder="开户行及账号"></el-input>
      </el-form-item>
      <el-form-item label="邮寄地址" prop="taxAddress">
        <el-input v-model="dataForm.taxAddress" placeholder="邮寄地址"></el-input>
      </el-form-item>
      <el-form-item label="确认是否已缴费" prop="isPay">
        <!-- <el-input v-model="dataForm.isPay" placeholder="确认是否已缴费"></el-input> -->
        <el-radio v-model="dataForm.isPay" :label="1">是</el-radio>
        <el-radio v-model="dataForm.isPay" :label="0">否</el-radio>
      </el-form-item>
      <!-- <el-form-item label="创建时间" prop="createTime">
        <el-input v-model="dataForm.createTime" placeholder="创建时间"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="是否被删除 状态  0：正常   1：删除" prop="isDel">
        <el-input v-model="dataForm.isDel" placeholder="是否被删除 状态  0：正常   1：删除"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="会议id" prop="meetingId">
        <el-input v-model="dataForm.meetingId" placeholder="会议id"></el-input>
      </el-form-item>-->
      <!-- <el-form-item label="发票类型0普1专" prop="taxType">
        <el-input v-model="dataForm.taxType" placeholder="发票类型0普1专"></el-input>
      </el-form-item>-->
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
        attenders: "", //参会人员id
        payTime: "", //付款时间
        payType: "", //付款方式
        payOption: "", //付款项目
        feeId: "", //费用类型
        money: "", //金额
        isPay: "", //是否付款
        taxType: "", //发票类型
        taxTitle: "", //发票抬头
        taxNumber: "", //发票号
        addrPhone: "", //电话
        bankAddrAccount: "", //开户行
        taxAddress: "" //发票地址
      },
      dataRule: {
        orderId: [
          { required: true, message: "订单号不能为空", trigger: "blur" }
        ],
        attendersId: [
          { required: true, message: "参会人员id不能为空", trigger: "blur" }
        ],
        feeId: [
          { required: true, message: "费用类型id不能为空", trigger: "blur" }
        ],
        payTime: [
          { required: true, message: "缴费时间不能为空", trigger: "blur" }
        ],
        payType: [
          { required: true, message: "缴费方式不能为空", trigger: "blur" }
        ],
        payOption: [
          { required: true, message: "缴费项目不能为空", trigger: "blur" }
        ],
        money: [{ required: true, message: "金额不能为空", trigger: "blur" }],
        taxTitle: [
          { required: true, message: "发票抬头不能为空", trigger: "blur" }
        ],
        taxNumber: [
          { required: true, message: "纳税人税号不能为空", trigger: "blur" }
        ],
        addrPhone: [
          { required: true, message: "地址 电话不能为空", trigger: "blur" }
        ],
        bankAddrAccount: [
          { required: true, message: "开户行及账号不能为空", trigger: "blur" }
        ],
        taxAddress: [
          { required: true, message: "邮寄地址不能为空", trigger: "blur" }
        ],
        isPay: [
          { required: true, message: "确认是否已缴费不能为空", trigger: "blur" }
        ],
        createTime: [
          { required: true, message: "创建时间不能为空", trigger: "blur" }
        ],

        meetingId: [
          { required: true, message: "会议id不能为空", trigger: "blur" }
        ],
        taxType: [
          { required: true, message: "发票类型0普1专不能为空", trigger: "blur" }
        ]
      },
      loading: false,
      meetingList: [],
      list: [],
      options: [],
      payTypeList: [
        {
          value: "1",
          label: "线上缴费"
        },
        {
          value: "2",
          label: "现场缴费"
        }
      ],
      typesoffeeList: [],
      option: [],
      feeIdTypeList: [], //费用类型
      option: [],
      page: 0,
      limit: 10,
      meetingId: this.$route.params.id
    };
  },
  created() {
    this.getPaymentByname();
    this.getTypesoffee();
  },
  methods: {
    init(id) {
      this.dataForm.id = id || 0;
      this.visible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].resetFields();
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/admin/payment/info/${this.dataForm.id}`),
            method: "get",
            params: this.$http.adornParams()
          }).then(({ data }) => {
            window.console.log(data);
            if (data && data.code === 0) {
              this.dataForm.orderId = data.payment.orderId;
              this.dataForm.attenders = data.payment.attenders;
              this.dataForm.feeId = data.payment.feeId;
              this.dataForm.payTime = data.payment.payTime;
              this.dataForm.payType = data.payment.payType;
              this.dataForm.payOption = data.payment.payOption;
              this.dataForm.money = data.payment.money;
              this.dataForm.taxTitle = data.payment.taxTitle;
              this.dataForm.taxNumber = data.payment.taxNumber;
              this.dataForm.addrPhone = data.payment.addrPhone;
              this.dataForm.bankAddrAccount = data.payment.bankAddrAccount;
              this.dataForm.taxAddress = data.payment.taxAddress;
              this.dataForm.isCheck = data.payment.isCheck;
              this.dataForm.createTime = data.payment.createTime;
              this.dataForm.isPay = data.payment.isPay;
              this.dataForm.meetingId = data.payment.meetingId;
              this.dataForm.taxType = data.payment.taxType;
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
              `/admin/payment/${!this.dataForm.id ? "save" : "update"}`
            ),
            method: "post",
            data: this.$http.adornData({
              id: this.dataForm.id || undefined,
              attenders: this.dataForm.attenders,
              feeId: this.dataForm.feeId,
              payTime: this.dataForm.payTime,
              payType: this.dataForm.payType,
              payOption: this.dataForm.payOption,
              money: this.dataForm.money,
              taxTitle: this.dataForm.taxTitle,
              taxNumber: this.dataForm.taxNumber,
              addrPhone: this.dataForm.addrPhone,
              bankAddrAccount: this.dataForm.bankAddrAccount,
              taxAddress: this.dataForm.taxAddress,
              isPay: this.dataForm.isPay,
              createTime: this.dataForm.createTime,
              meetingId: this.dataForm.meetingId,
              taxType: this.dataForm.taxType
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
    getPaymentByname(name = "") {
      this.loading = true;
      this.$http({
        url: this.$http.adornUrl(`/admin/attenders/selectbyname`),
        method: "get",
        params: this.$http.adornParams({
          name: name,
          meetingId: this.$route.params.id
        })
      }).then(({ data }) => {
        if (data) {
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
    },
    getTypesoffee() {
      this.$http({
        url: this.$http.adornUrl(
          `/admin/typesoffee/listbymid/${this.meetingId}`
        ),
        methods: "get",
        params: {}
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.typesoffeeList = data.list;
          this.option = this.typesoffeeList.map(item => {
            return {
              value: item.id,
              label: item.name
            };
          });
        }
      });
    }
  }
};
</script>
