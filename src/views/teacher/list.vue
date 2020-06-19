<template>
  <div class="app-container">
    <el-form :inline="true">
      <el-form-item>
        <!-- <el-input v-model="searchObj.name" placeholder="讲师"/> -->
        <el-autocomplete
          v-model="searchObj.name"
          :fetch-suggestions="querySearch"
          :trigger-on-focus="false"
          value-key="name"
          class="inline-input"
          placeholder="讲师" />
      </el-form-item>
      <el-form-item>
        <el-input v-model="searchObj.level" placeholder="头衔">
          <el-option value="1" label="高级讲师"/>
          <el-option value="2" label="首席讲师"/>
        </el-input>
      </el-form-item>
      <el-form-item label="入驻时间">
        <el-date-picker
          v-model="searchObj.joinDateBegin"
          placeholder="开始时间"
          value-format="yyyy-MM-dd"/>
      </el-form-item>
      <el-form-item label="结束时间">
        <el-date-picker
          v-model="searchObj.joinDateEnd"
          placeholder="结束时间"
          value-format="yyyy-MM-dd"/>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icom="el-icon-search" @click="fetchData()">查询</el-button>
        <el-button type="defualt" @click="resetData()">清空</el-button>
      </el-form-item>
    </el-form>
    <div style="margin-bottom: 10px">
      <el-button type="danger" size="mini" @click="batchRemove()">批量删除</el-button>
    </div>

    <el-table
      :data="list"
      border
      stripe
      @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="70"/>
      <el-table-column
        label="#"
        width="50">
        <template slot-scope="scope">
          {{ (page - 1) * limit + scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column prop="name" label="名称" width="70"/>
      <el-table-column prop="level" label="头衔" width="95" >
        <template slot-scope="scope">
          <el-tag v-if="scope.row.level === 1" type="success">高级讲师</el-tag>
          <el-tag v-if="scope.row.level === 2">首席讲师</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="intro" label="资历"/>
      <el-table-column prop="sort" label="排序" width="100"/>
      <el-table-column prop="joinDate" label="入驻时间" width="100"/>
      <el-table-column label="操作" width="200">
        <template slot-scope="scope">
          <router-link :to="'/teacher/edit/' + scope.row.id">
            <el-button type="primary" size="mini" icon="el-icon-edit">修改</el-button>
          </router-link>
          <el-button
            size="mini"
            type="danger"
            @click="removeById(scope.row.id)">删除</el-button>

        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <el-pagination
      :total="total"
      :current-page="page"
      :page-size="limit"
      :page-sizes="[5,
                    10,
                    20]"
      style="padding: 30px; text-align: center"
      layout="sizes, prev, pager, next, jumper, ->, total"
      @size-change="changePageSize"
      @current-change="changeCurrentPage" />
  </div>
</template>

<script>
import teacherApi from '@/api/teacher'
export default {
  data() {
    return {
      list: [], // 讲师列表
      total: 0,
      page: 1, // 页码
      limit: 5,
      searchObj: {},
      multipleSelection: []
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    // diaoyong api 加载讲师列表
    fetchData() {
      teacherApi.pageList(this.page, this.limit, this.searchObj).then(respnese => { // 此处rep已经由响应拦截器处理过
        this.list = respnese.data.rows
        this.total = respnese.data.total
      })
    },

    changeCurrentPage(page) {
      this.page = page
      this.fetchData()
    },

    changePageSize(size) {
      this.limit = size
      this.fetchData()
    },

    resetData() {
      this.searchObj = {}
      this.fetchData()
    },

    // 删除记录
    removeById(id) {
      // 询问是否删除
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        return teacherApi.removeById(id)
      }).then(response => {
        // 刷新页面
        this.fetchData()
        // 弹出成功提示
        this.$message({
          message: response.message,
          type: 'success'
        })
      }).catch((err) => {
        console.log(err)
        if (err === 'cancel') {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        }
      })
    },

    batchRemove() {
      // 判断数据是否选中
      if (this.multipleSelection.length === 0) {
        this.$message({
          type: 'warning',
          message: '请选择删除项'
        })
        return
      }
      // 确认
      this.$confirm('此操作永久删除选中数据，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          const idList = []
          this.multipleSelection.forEach(element => {
            idList.push(element.id)
          })
          teacherApi.batchRemove(idList).then(response => {
            this.fetchData()
            this.$message({
              message: response.message,
              type: 'success'
            })
          })
          // 刷新
        })
        .catch((err) => {
          if (err === 'cancel') {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          }
        })
    },

    handleSelectionChange(selection) {
      this.multipleSelection = selection
    },

    // 输入建议
    querySearch(queryString, callback) {
      teacherApi.selectNameListByKey(queryString).then(res => {
        console.log(res.data.nameList)
        callback(res.data.nameList)
      })
    }
  }
}

</script>
