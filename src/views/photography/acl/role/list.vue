<template>
  <div class="app-container">

    <!--查询表单-->
    <el-card class="filter-container" shadow="never">
      <div>
        <i class="el-icon-search" />
        <span>筛选搜索</span>
      </div>

      <div style="margin-top: 20px">
        <el-form :inline="true" class="demo-form-inline">
          <el-form-item>
            <el-input v-model="searchObj.roleName" placeholder="角色名称" />
          </el-form-item>

          <el-button type="primary" icon="el-icon-search" @click="fetchData()">查询</el-button>
          <el-button type="default" @click="resetData()">清空</el-button>
          <el-button type="primary" @click="addRole()">添加</el-button>
          <el-button type="danger" @click="removeRows()">批量删除</el-button>
        </el-form>
      </div>
    </el-card>

    <!-- 讲师列表 -->
    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="数据加载中"
      border
      fit
      highlight-current-row
      style="margin-top: 20px"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="序号" width="70" align="center">
        <template slot-scope="scope">
          {{ (page - 1) * limit + scope.$index + 1 }}
        </template>
      </el-table-column>
      <el-table-column prop="roleName" label="角色名称" width="100" align="center" />
      <el-table-column prop="remark" label="备注" align="center" />
      <el-table-column prop="gmtCreate" label="创建时间" width="180" align="center" />
      <el-table-column label="操作" width="280" align="center">
        <template slot-scope="scope">
          <router-link :to="'/acl/role/distribution/'+scope.row.id" style="padding-right: 5px">
            <el-button type="info" size="mini" icon="el-icon-info">权限</el-button>
          </router-link>
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="updateById(scope.row.id)">修改</el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页组件 -->
    <el-pagination
      :current-page="page"
      :total="total"
      :page-size="limit"
      :page-sizes="[5, 10, 20, 30, 40, 50, 100]"
      style="padding: 30px 0; text-align: center;"
      layout="total, prev, pager, next, jumper"
      @current-change="fetchData"
      @size-change="changeSize"
    />

    <el-dialog title="新增角色" :visible.sync="dialogFormVisible" @close="closeDialog">
      <el-form ref="user" :model="role" :rules="validateRules" label-width="120px">
        <el-form-item label="角色名" prop="username">
          <el-input v-model="role.roleName" />
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="role.remark" />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="save">保存</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>

    <el-dialog title="修改角色" :visible.sync="dialogFormVisible1" @close="closeDialog">
      <el-form ref="role" :model="role" :rules="validateRules" label-width="120px">
        <el-form-item label="角色名" prop="roleName">
          <el-input v-model="role.roleName" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="role.remark" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="update">保存</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>

  </div>
</template>

<script>
import roleApi from '@/api/acl/role'

export default {
  data() {
    return {
      listLoading: true, // 数据是否正在加载
      list: null, // 讲师列表
      total: 0, // 数据库中的总记录数
      page: 1, // 默认页码
      limit: 10, // 每页记录数
      searchObj: {}, // 查询表单对象
      multipleSelection: [], // 批量选择中选择的记录列表
      dialogFormVisible: false,
      dialogFormVisible1: false,
      role: {},
      validateRules: {
        roleName: [
          { required: true, trigger: 'blur', message: '请输入角色名' }
        ],
        remark: [
          { required: true, trigger: 'blur', message: '请输入备注' }
        ]
      }
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    update() {
      // teacher数据的获取
      roleApi.updateById(this.role)
        .then(response => {
          if (response.success) {
            this.$message({
              type: 'success',
              message: response.message
            })
            this.dialogFormVisible1 = false
            this.fetchData()
          }
        })
    },
    updateById(id) {
      roleApi.getRoleById(id)
        .then(response => {
          this.role = response.data.item
        })
      this.dialogFormVisible1 = true
    },
    save() {
      roleApi.save(this.role)
        .then(response => {
          if (response.success) {
            this.$message({
              type: 'success',
              message: response.message
            })
            this.dialogFormVisible = false
            this.fetchData()
          }
        })
    },
    closeDialog() {
      this.role = {}
      this.$nextTick(() => {
        this.$refs.role.clearValidate()
      })
    },
    // 当页码发生改变的时候
    changeSize(size) {
      this.limit = size
      this.fetchData(1)
    },

    addRole() {
      this.dialogFormVisible = true
    },

    // 加载讲师列表数据
    fetchData(page = 1) {
      // 异步获取远程数据（ajax）
      this.page = page

      roleApi.getPageList(this.page, this.limit, this.searchObj).then(
        response => {
          this.list = response.data.items
          this.total = response.data.total

          // 数据加载并绑定成功
          this.listLoading = false
        }
      )
    },

    // 重置查询表单
    resetData() {
      this.searchObj = {}
      this.fetchData()
    },

    // 根据id删除数据
    removeDataById(id) {
      // debugger
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { // promise
        // 点击确定，远程调用ajax
        return roleApi.removeById(id)
      }).then((response) => {
        this.fetchData(this.page)
        if (response.success) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    // 当表格复选框选项发生变化的时候触发
    handleSelectionChange(selection) {
      this.multipleSelection = selection
    },

    // 批量删除
    removeRows() {
      if (this.multipleSelection.length === 0) {
        this.$message({
          type: 'warning',
          message: '请选择要删除的记录!'
        })
        return
      }
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { // promise
        // 点击确定，远程调用ajax
        // 遍历selection，将id取出放入id列表
        var idList = []
        this.multipleSelection.forEach(item => {
          idList.push(item.id)
        })
        // 调用api
        return roleApi.removeRows(idList)
      }).then((response) => {
        this.fetchData(this.page)
        if (response.success) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

