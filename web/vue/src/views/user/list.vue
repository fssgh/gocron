<template>
  <el-card>
    <template #header>
      <div class="card-header">
        <strong>用户列表</strong>
      </div>
    </template>
    <el-row type="flex" justify="end">
      <el-col :span="2">
        <el-button type="primary" @click="toEdit(null)">新增</el-button>
      </el-col>
      <el-col :span="2">
        <el-button type="info" @click="refresh">刷新</el-button>
      </el-col>
    </el-row>
    <el-pagination
        background
        layout="prev, pager, next, sizes, total"
        :total="userTotal"
        :page-size="20"
        @size-change="changePageSize"
        @current-change="changePage"
        @prev-click="changePage"
        @next-click="changePage">
    </el-pagination>
    <el-table
        :data="users"
        tooltip-effect="dark"
        border
        v-loading="loading"
        style="width: 100%">
      <el-table-column
          prop="id"
          label="用户id">
      </el-table-column>
      <el-table-column
          prop="name"
          label="用户名">
      </el-table-column>
      <el-table-column prop="source" label="来源"/>
      <el-table-column
          prop="email"
          label="邮箱">
      </el-table-column>
      <el-table-column
          prop="is_admin"
          :formatter="formatRole"
          label="角色">
      </el-table-column>
      <el-table-column
          label="状态">
        <template #default="scope">
          <el-switch
              v-model="scope.row.status"
              :active-value="1"
              :inactive-value="0"
              active-color="#13ce66"
              @change="changeStatus(scope.row)"
              inactive-color="#ff4949">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="300" v-if="this.isAdmin">
        <template #default="scope">
          <el-row>
            <el-button type="primary" @click="toEdit(scope.row)">编辑</el-button>
            <el-button type="success" @click="editPassword(scope.row)">修改密码</el-button>
            <el-button type="danger" @click="remove(scope.row)">删除</el-button>
          </el-row>
          <br>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
</template>

<script>
import userService from '../../api/user'

export default {
  name: 'user-list',
  data() {
    return {
      loading: false,
      users: [],
      userTotal: 0,
      searchParams: {
        page_size: 20,
        page: 1
      },
      isAdmin: this.$store.getters.user.isAdmin
    }
  },
  created() {
    this.search()
  },
  methods: {
    changeStatus(item) {
      if (item.status) {
        userService.enable(item.id)
      } else {
        userService.disable(item.id)
      }
    },
    formatRole(row, col) {
      if (row[col.property] === 1) {
        return '管理员'
      }
      return '普通用户'
    },
    changePage(page) {
      this.searchParams.page = page
      this.search()
    },
    changePageSize(pageSize) {
      this.searchParams.page_size = pageSize
      this.search()
    },
    search(callback = null) {
      let _this = this;
      _this.loading = true;
      userService.list(this.searchParams, (data) => {
        _this.loading = false;
        this.users = data.data
        this.userTotal = data.total
        if (callback) {
          callback()
        }
      })
    },
    remove(item) {
      this.$appConfirm(() => {
        userService.remove(item.id, () => {
          this.refresh()
        })
      })
    },
    toEdit(item) {
      let path = ''
      if (item === null) {
        path = '/user/create'
      } else {
        path = `/user/edit/${item.id}`
      }
      this.$router.push(path)
    },
    refresh() {
      this.search(() => {
        this.$message.success('刷新成功')
      })
    },
    editPassword(item) {
      this.$router.push(`/user/edit-password/${item.id}`)
    }
  }
}
</script>
