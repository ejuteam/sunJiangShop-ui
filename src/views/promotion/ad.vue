<template>
  <div class="app-container">

    <!-- 查询和其他操作 -->
    <div class="filter-container">
      <el-input
        v-model="listQuery.name"
        clearable
        size="mini"
        class="filter-item"
        style="width: 200px;"
        placeholder="请输入广告标题"
      />
      <el-input
        v-model="listQuery.content"
        clearable
        size="mini"
        class="filter-item"
        style="width: 200px;"
        placeholder="请输入广告内容"
      />
      <el-button
        v-permission="['GET /admin/ad/list']"
        size="mini"
        class="filter-item"
        type="primary"
        icon="el-icon-search"
        @click="handleFilter"
      >查找</el-button>
      <el-button
        v-permission="['POST /admin/ad/create']"
        size="mini"
        class="filter-item"
        type="primary"
        icon="el-icon-edit"
        @click="handleCreate"
      >添加</el-button>
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

      <!--      <el-table-column align="center" label="广告ID" prop="id" sortable/>

      <el-table-column align="center" label="广告标题" prop="name"/>

      <el-table-column align="center" label="广告内容" prop="content"/>-->

      <el-table-column
        align="center"
        label="关联商品"
        prop="goodsName"
        width="350"
      />
      <el-table-column
        align="center"
        label="图片"
        prop="url"
      >
        <template slot-scope="scope">
          <img
            v-if="scope.row.url"
            :src="scope.row.url"
            width="80"
          >
        </template>
      </el-table-column>

      <!--
      <el-table-column align="center" label="轮播图位置" prop="position"/>
-->

      <!--
      <el-table-column align="center" label="活动链接" prop="link"/>
-->

      <el-table-column
        align="center"
        label="是否启用"
        prop="enabled"
      >
        <template slot-scope="scope">
          <el-tag :type="scope.row.enabled ? 'success' : 'error' ">{{ scope.row.enabled ? '启用' : '不启用' }}</el-tag>
        </template>
      </el-table-column>

      <el-table-column
        align="center"
        label="操作"
        width="200"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="scope">
          <el-button
            v-permission="['POST /admin/ad/update']"
            type="primary"
            size="mini"
            @click="handleUpdate(scope.row)"
          >编辑</el-button>
          <el-button
            v-permission="['POST /admin/ad/delete']"
            type="danger"
            size="mini"
            @click="handleDelete(scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total>0"
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
        style="width: 400px; margin-left:50px;"
      >
        <!--        <el-form-item label="广告标题" prop="name">
          <el-input v-model="dataForm.name"/>
        </el-form-item>
        <el-form-item label="广告内容" prop="content">
          <el-input v-model="dataForm.content"/>
        </el-form-item>-->
        <el-form-item
          label="广告图片"
          prop="url"
        >
          <el-upload
            :headers="headers"
            :action="uploadPath"
            :show-file-list="false"
            :on-success="uploadUrl"
            class="avatar-uploader"
            accept=".jpg,.jpeg,.png,.gif"
          >
            <img
              v-if="dataForm.url"
              :src="dataForm.url"
              class="avatar"
            >
            <i
              v-else
              class="el-icon-plus avatar-uploader-icon"
            />
          </el-upload>
        </el-form-item>
        <el-form-item
          label="关联商品"
          prop="goodsId"
        >
          <el-select
            v-model="dataForm.goodsId"
            filterable
            remote
            reserve-keyword
            placeholder="选择关联商品"
            :remote-method="fetchGoodsOptions"
            :loading="goodsLoading"
            clearable
          >
            <el-option
              v-for="item in goodsOptions"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <!--        <el-form-item label="轮播图位置" prop="position">
                  <el-select v-model="dataForm.position" placeholder="请选择">
                    <el-option :value=true label="首页"/>
                  </el-select>
                </el-form-item>
                <el-form-item label="活动链接" prop="link">
                  <el-input v-model="dataForm.link"/>
                </el-form-item>-->
        <el-form-item
          label="是否启用"
          prop="enabled"
        >
          <el-select
            v-model="dataForm.enabled"
            placeholder="请选择"
          >
            <el-option
              :value="true"
              label="启用"
            />
            <el-option
              :value="false"
              label="不启用"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="dialogFormVisible = false">取消</el-button>
        <el-button
          v-if="dialogStatus=='create'"
          type="primary"
          @click="createData"
        >确定</el-button>
        <el-button
          v-else
          type="primary"
          @click="updateData"
        >确定</el-button>
      </div>
    </el-dialog>
    <el-dialog
      title="确认删除"
      :visible.sync="deleteDialogVisible"
      width="30%"
      :before-close="handleCloseDeleteDialog"
    >
      <span>确定要删除该轮播图吗？</span>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="deleteDialogVisible = false">取消</el-button>
        <el-button
          type="danger"
          @click="confirmDelete"
        >删除</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { listAd, createAd, updateAd, deleteAd } from '@/api/business/ad'
import { uploadPath } from '@/api/business/storage'
import { getToken } from '@/utils/auth'
import Pagination from '@/components/Pagination'
import { listGoods } from '@/api/business/goods' // Secondary package based on el-pagination

export default {
  name: 'Ad',
  components: { Pagination },
  data() {
    return {
      uploadPath,
      list: undefined,
      total: 0,
      listLoading: true,
      goodsOptions: [], // 商品列表
      goodsMap: {}, // 商品ID到名称的映射
      goodsLoading: false,
      listQuery: {
        page: 1,
        limit: 20,
        name: undefined,
        content: undefined,
        sort: 'add_time',
        order: 'desc'
      },
      dataForm: {
        id: undefined,
        name: undefined,
        content: undefined,
        url: undefined,
        link: undefined,
        position: 1,
        enabled: true
      },
      listGoodsQuery: {
        page: 1,
        limit: 20,
        goodsSn: undefined,
        name: undefined,
        sort: 'add_time',
        order: 'desc'
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑',
        create: '创建'
      },
      rules: {/*
        name: [
          { required: true, message: '广告标题不能为空', trigger: 'blur' }
        ],
        content: [
          { required: true, message: '广告内容不能为空', trigger: 'blur' }
        ],
        url: [{ required: true, message: '广告链接不能为空', trigger: 'blur' }]*/
      },
      downloadLoading: false,
      deleteDialogVisible: false, // 控制删除确认弹框的显示
      deleteItem: null // 存储要删除的广告信息
    }
  },
  computed: {
    headers() {
      return {
        'X-Dts-Admin-Token': getToken()
      }
    }
  },
  created() {
    this.getList()
    this.fetchGoodsOptions()
  },
  methods: {

    getList() {
      this.listLoading = true
      listAd(this.listQuery)
        .then(response => {
          this.list = response.data.data.items.map(ad => {
            return {
              ...ad,
              goodsName: this.goodsMap[ad.goodsId] || '未知商品' // 根据ID回显商品名称
            }
          })
          this.total = response.data.data.total
          this.listLoading = false
        })
        .catch(() => {
          this.list = []
          this.total = 0
          this.listLoading = false
        })
    },
    fetchGoodsOptions(query) {
      this.goodsLoading = true
      listGoods({ name: query, page: 1, limit: 10 })
        .then(response => {
          this.goodsOptions = response.data.data.items || []
          this.goodsOptions.forEach(item => {
            this.goodsMap[item.id] = item.name // 生成ID到名称的映射
          })
          this.goodsLoading = false
        })
        .catch(error => {
          console.error('Error fetching goods options:', error)
          this.goodsOptions = []
          this.goodsLoading = false
        })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    resetForm() {
      this.dataForm = {
        id: undefined,
        name: undefined,
        content: undefined,
        url: undefined,
        link: undefined,
        position: 1,
        enabled: true,
        goodsId: ''
      }
    },
    handleCreate() {
      this.resetForm()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    uploadUrl: function(response) {
      this.dataForm.url = response.data.url
    },
    createData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          createAd(this.dataForm)
            .then(response => {
              this.list.unshift(response.data.data)
              this.dialogFormVisible = false
              this.$notify.success({
                title: '成功',
                message: '创建成功'
              })
            })
            .catch(response => {
              this.$notify.error({
                title: '失败',
                message: response.data.errmsg
              })
            })
        }
      })
    },
    handleUpdate(row) {
      this.dataForm = Object.assign({}, row)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate(valid => {
        if (valid) {
          updateAd(this.dataForm)
            .then(() => {
              for (const v of this.list) {
                if (v.id === this.dataForm.id) {
                  const index = this.list.indexOf(v)
                  this.list.splice(index, 1, this.dataForm)
                  break
                }
              }
              this.dialogFormVisible = false
              this.$notify.success({
                title: '成功',
                message: '更新广告成功'
              })
            })
            .catch(response => {
              this.$notify.error({
                title: '失败',
                message: response.data.errmsg
              })
            })
        }
      })
    },
    handleDelete(row) {
      this.deleteItem = row // 暂存要删除的广告
      this.deleteDialogVisible = true // 显示确认删除弹框
      /*
      deleteAd(row)
        .then(response => {
          this.$notify.success({
            title: '成功',
            message: '删除成功'
          })
          const index = this.list.indexOf(row)
          this.list.splice(index, 1)
        })
        .catch(response => {
          this.$notify.error({
            title: '失败',
            message: response.data.errmsg
          })
        })*/
    },
    // 确认删除操作
    confirmDelete() {
      deleteAd(this.deleteItem)
        .then(() => {
          this.$notify.success({
            title: '成功',
            message: '删除成功'
          })
          const index = this.list.indexOf(this.deleteItem)
          this.list.splice(index, 1)
          this.deleteDialogVisible = false // 关闭弹框
          this.deleteItem = null // 清空暂存的广告信息
        })
        .catch(error => {
          this.$notify.error({
            title: '失败',
            message: error.data.errmsg
          })
        })
    },
    // 关闭删除确认弹框时清空删除对象
    handleCloseDeleteDialog() {
      this.deleteDialogVisible = false
      this.deleteItem = null
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = [
          '广告ID',
          '广告标题',
          '广告内容',
          '广告图片',
          '广告位置',
          '活动链接',
          '是否启用'
        ]
        const filterVal = [
          'id',
          'name',
          'content',
          'url',
          'postion',
          'link',
          'enabled'
        ]
        excel.export_json_to_excel2(tHeader, this.list, filterVal, '广告信息')
        this.downloadLoading = false
      })
    }
  }
}
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #20a0ff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 120px;
  height: 120px;
  line-height: 120px;
  text-align: center;
}
.avatar {
  width: 145px;
  height: 145px;
  display: block;
}
</style>
