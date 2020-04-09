<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('admin:meeting:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('admin:meeting:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="会议id (会议表)">
      </el-table-column>
      <el-table-column
        prop="companyId"
        header-align="center"
        align="center"
        label="举办单位id">
      </el-table-column>
      <el-table-column
        prop="nameCn"
        header-align="center"
        align="center"
        label="会议名称(中文)">
      </el-table-column>
      <el-table-column
        prop="nameEn"
        header-align="center"
        align="center"
        label="会议名称(英文)">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="举办地点">
      </el-table-column>
      <el-table-column
        prop="startTime"
        header-align="center"
        align="center"
        label="开始时间">
      </el-table-column>
      <el-table-column
        prop="endTime"
        header-align="center"
        align="center"
        label="结束时间">
      </el-table-column>
      <el-table-column
        prop="officeWebsite"
        header-align="center"
        align="center"
        label="官方网址">
      </el-table-column>
      <el-table-column
        prop="titlePicture"
        header-align="center"
        align="center"
        label="会议封面图片地址">
      </el-table-column>
      <el-table-column
        prop="subjects"
        header-align="center"
        align="center"
        label="涉及学科">
      </el-table-column>
      <el-table-column
        prop="industries"
        header-align="center"
        align="center"
        label="涉及行业">
      </el-table-column>
      <el-table-column
        prop="attendersType"
        header-align="center"
        align="center"
        label="参会人员类型">
      </el-table-column>
      <el-table-column
        prop="serviceEmp"
        header-align="center"
        align="center"
        label="会议负责人">
      </el-table-column>
      <el-table-column
        prop="onlineRegDeadline"
        header-align="center"
        align="center"
        label="线上报名截止时间">
      </el-table-column>
      <el-table-column
        prop="onsiteRegDeadline"
        header-align="center"
        align="center"
        label="现场报名截止时间">
      </el-table-column>
      <el-table-column
        prop="subDeadline"
        header-align="center"
        align="center"
        label="投稿截止时间">
      </el-table-column>
      <el-table-column
        prop="subRequirement"
        header-align="center"
        align="center"
        label="投稿范围及要求">
      </el-table-column>
      <el-table-column
        prop="introduction"
        header-align="center"
        align="center"
        label="会议介绍">
      </el-table-column>
      <el-table-column
        prop="paperRequireId"
        header-align="center"
        align="center"
        label="投稿要求id">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="modifyTime"
        header-align="center"
        align="center"
        label="修改时间">
      </el-table-column>
      <el-table-column
        prop="isCheck"
        header-align="center"
        align="center"
        label="是否通过审核 0:未通过 1:通过">
      </el-table-column>
      <el-table-column
        prop="isDel"
        header-align="center"
        align="center"
        label="是否被删除 状态  0：正常   1：删除">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './meeting-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/admin/meeting/list'),
          method: 'get',
          params: this.$http.adornParams({
            // 'page': this.pageIndex,
            // 'limit': this.pageSize,
            // 'key': this.dataForm.key
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.list
            // this.dataList = data.page.list
            // this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/admin/meeting/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }
</script>
