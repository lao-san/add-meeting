<style>
ul li {
  list-style-type: none
}
</style>
<template>
  <div class="mod-config">
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>未发布中的会议</h2>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="6">
        <div class="grid-content bg-purple-light">
          <el-button
            v-if="isAuth('admin:meeting:save')"
            type="primary"
            @click="addOrUpdateHandle()"
          >+ 创建新会议</el-button>
        </div>
      </el-col>
      <ul>
        <template v-for="item in dataList">
          <li :key="item.id" v-if="!item.isCheck" @click="routerTo(item.id)">
              <el-col :span="6">
                <div class="grid-content bg-purple-dark">
                  <img src="/static/img/huiyiim.jpg" alt="会议图片" />
                  <h3>{{item.nameCn}}</h3>
                  <div>
                    <span>{{item.address}}</span>
                    <span>{{item.startTime}}</span>
                  </div>
                </div>
              </el-col>
          </li>
        </template>
      </ul>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>发布中的会议</h2>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <ul>
        <template v-for="item in dataList">
          <li :key="item.id" v-if="item.isCheck == 1" @click="routerTo(item.id)">
              <el-col :span="6">
                <div class="grid-content bg-purple-dark">
                  <img src="/static/img/huiyiim.jpg" alt="会议图片" />
                  <h3>{{item.nameCn}}</h3>
                  <div>
                    <span>{{item.address}}</span>
                    <span>{{item.startTime}}</span>
                  </div>
                </div>
              </el-col>
          </li>
        </template>
      </ul>
    </el-row>
    <el-row>
      <el-col :span="24">
        <div class="grid-content bg-purple-dark">
          <h2>已结束的会议</h2>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <ul>
        <template v-for="item in dataList">
          <li :key="item.id" v-if="item.isCheck == 2" @click="routerTo(item.id)">
              <el-col :span="6">
                <div class="grid-content bg-purple-dark">
                  <img src="/static/img/huiyiim.jpg" alt="会议图片" />
                  <h3>{{item.nameCn}}</h3>
                  <div>
                    <span>{{item.address}}</span>
                    <span>{{item.startTime}}</span>
                  </div>
                </div>
              </el-col>
          </li>
        </template>
      </ul>
    </el-row>

    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
import AddOrUpdate from '../modules/admin/meeting-add-or-update'
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
    this.meetingMenuId = 0
    this.getDataList()
  },
  methods: {
    routerTo (mid) {
      this.$router.push({ name: 'admin-meeting/:id', params: { id: mid } })
    },
    // 获取数据列表
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/admin/meeting/list'),
        method: 'get',
        params: this.$http.adornParams({
          // page: this.pageIndex,
          // limit: this.pageSize,
          // key: this.dataForm.key
        })
      }).then(({ data }) => {
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
      var ids = id
        ? [id]
        : this.dataListSelections.map(item => {
          return item.id
        })
      this.$confirm(
        `确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(() => {
        this.$http({
          url: this.$http.adornUrl('/admin/meeting/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
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
