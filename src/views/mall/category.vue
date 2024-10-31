<template>
  <div class="app-container">

    <!-- 查询和其他操作 -->
    <div class="toolbar">
      <el-card class="query-card">
        <el-form
          :inline="true"
          label-width="80px"
        >
          <el-form-item label="分类ID">
            <el-input
              v-model="listQuery.id"
              clearable
              size="mini"
              class="filter-item"
              style="width: 200px;"
              placeholder="请输入分类ID"
            />
          </el-form-item>
          <el-form-item label="分类名称">
            <el-input
              v-model="listQuery.name"
              clearable
              size="mini"
              class="filter-item"
              style="width: 200px;"
              placeholder="请输入分类名称"
            />
          </el-form-item>
          <el-form-item
            class="query-buttons"
            style="margin-left: auto;"
          >
            <el-button
              v-permission="['GET /admin/category/list']"
              size="mini"
              class="filter-item"
              type="primary"
              icon="el-icon-search"
              @click="handleFilter"
            >
              查找
            </el-button>
            <el-button
              v-permission="['POST /admin/category/create']"
              size="mini"
              class="filter-item"
              type="primary"
              icon="el-icon-edit"
              @click="handleCreate"
            >
              新增分类
            </el-button>
            <el-button
              size="mini"
              class="filter-item"
              icon="el-icon-refresh"
              @click="resetFilter"
            >重置
            </el-button>
          </el-form-item>
        </el-form>
      </el-card>
    </div>

    <!-- 查询结果 -->
    <el-table
      v-loading="listLoading"
      :data="list"
      size="small"
      element-loading-text="正在查询中。。。"
      border
      fit
      highlight-current-row
    >
      <el-table-column
        align="center"
        label="分类ID"
        prop="id"
        sortable
      />
      <el-table-column
        align="center"
        label="分类名称"
        prop="name"
      />
      <el-table-column
        align="center"
        label="商品数量"
        prop="goodsNum"
      />
      <el-table-column
        align="center"
        label="操作"
        width="200"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="scope">
          <el-button
            v-permission="['POST /admin/category/update']"
            type="primary"
            size="mini"
            @click="handleUpdate(scope.row)"
          >
            编辑
          </el-button>
          <el-button
            v-permission="['POST /admin/category/delete']"
            type="danger"
            size="mini"
            @click="handleDelete(scope.row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />

    <!-- 添加或修改对话框 -->
    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="dataForm"
        status-icon
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left: 50px;"
      >
        <el-form-item
          label="分类名称"
          prop="name"
        >
          <el-input v-model="dataForm.name" />
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button
          v-if="dialogStatus == 'create'"
          type="primary"
          @click="createData"
        >
          确定
        </el-button>
        <el-button
          v-else
          type="primary"
          @click="updateData"
        >确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listCategory,
  createCategory,
  updateCategory,
  deleteCategory
} from '@/api/business/category'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

export default {
  name: 'Category',
  components: { Pagination },
  data() {
    return {
      list: undefined,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        id: undefined,
        name: undefined
      },
      dataForm: {
        id: undefined,
        name: ''
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '创建'
      },
      rules: {
        name: [{ required: true, message: '分类名不能为空', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      listCategory(this.listQuery)
        .then(response => {
          this.list = response.data.data.items
          this.total = response.data.data.total
          this.listLoading = false
        })
        .catch(() => {
          this.list = []
          this.total = 0
          this.listLoading = false
        })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    resetFilter() {
      this.listQuery = {
        page: 1,
        limit: 20,
        id: undefined,
        name: undefined
      }
      this.getList()
    },
    handleCreate() {
      this.resetForm()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    resetForm() {
      this.dataForm = {
        id: undefined,
        name: ''
      }
    },
    handleUpdate(row) {
      this.dataForm = Object.assign({}, row)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          createCategory(this.dataForm).then(response => {
            this.list.unshift(response.data.data)
            this.dialogFormVisible = false
            this.$notify.success({
              title: '成功',
              message: '创建成功'
            })
          })
        }
      })
    },
    updateData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          updateCategory(this.dataForm).then(() => {
            const index = this.list.findIndex(v => v.id === this.dataForm.id)
            if (index !== -1) this.list.splice(index, 1, this.dataForm)
            this.dialogFormVisible = false
            this.$notify.success({
              title: '成功',
              message: '更新成功'
            })
          })
        }
      })
    },
    handleDelete(row) {
      deleteCategory(row).then(() => {
        const index = this.list.indexOf(row)
        if (index > -1) this.list.splice(index, 1)
        this.$notify.success({
          title: '成功',
          message: '删除成功'
        })
      })
    }
  }
}
</script>

<style>
.filter-container {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 15px;
}
</style>
