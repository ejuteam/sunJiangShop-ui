<template>
  <div class="app-container">

    <!-- 查询和其他操作 -->
    <div class="toolbar">
      <el-card class="query-card">
        <el-form
          :inline="true"
          label-width="80px"
        >
          <el-form-item label="订单编号">
            <el-input
              v-model="listQuery.orderSn"
              clearable
              size="mini"
              class="filter-item"
              style="width: 200px;"
              placeholder="请输入订单编号"
            />
          </el-form-item>
          <el-form-item label="订单状态">
            <el-select
              v-model="listQuery.orderStatusArray"
              multiple
              size="mini"
              style="width: 200px"
              class="filter-item"
              placeholder="请选择订单状态"
            >
              <el-option
                v-for="(statuses, label) in statusMap"
                :key="label"
                :label="label"
                :value="statuses"
              />
            </el-select>
          </el-form-item>
          <el-form-item label="开始时间">
            <el-date-picker
              v-model="listQuery.payStartDate"
              type="date"
              placeholder="支付开始时间"
              size="mini"
              class="filter-item"
              style="width: 250px;"
              value-format="yyyyMMdd"
            />
          </el-form-item>
          <el-form-item label="结束时间">
            <el-date-picker
              v-model="listQuery.payEndDate"
              type="date"
              placeholder="支付结束时间"
              size="mini"
              class="filter-item"
              style="width: 250px;"
              value-format="yyyyMMdd"
            />
          </el-form-item>
          <el-form-item
            class="query-buttons"
            style="margin-left: auto;"
          >
            <el-button
              v-permission="['GET /admin/order/list']"
              size="mini"
              class="filter-item"
              type="primary"
              icon="el-icon-search"
              @click="handleFilter"
            >查找</el-button>
            <el-button
              size="mini"
              class="filter-item"
              icon="el-icon-refresh"
              @click="resetFilters"
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
        min-width="100"
        label="订单编号"
        prop="orderSn"
        sortable
      />

      <!--      <el-table-column
        align="center"
        min-width="100px"
        label="用户ID"
        prop="userId"
      />-->

      <el-table-column
        align="center"
        min-width="100px"
        label="订单状态"
        prop="orderStatus"
      >
        <template slot-scope="scope">
          <el-tag>{{ scope.row.orderStatus | orderStatusFilter }}</el-tag>
        </template>
      </el-table-column>

      <el-table-column
        align="center"
        min-width="100px"
        label="订单金额"
        prop="orderPrice"
      />

      <el-table-column
        align="center"
        min-width="100px"
        label="支付金额(含邮费)"
        prop="actualPrice"
      />
      <el-table-column
        align="center"
        min-width="100px"
        label="邮费"
        prop="freightPrice"
      />
      <el-table-column
        align="center"
        min-width="120px"
        label="支付时间"
        prop="payTime"
      />

      <el-table-column
        align="center"
        min-width="120px"
        label="物流单号"
        prop="shipSn"
      />

      <!--
      <el-table-column align="center" min-width="100px" label="物流渠道" prop="shipChannel"/>
-->

      <el-table-column
        align="center"
        label="操作"
        min-width="150px"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="scope">
          <el-button
            v-permission="['GET /admin/order/detail']"
            type="primary"
            size="mini"
            @click="handleDetail(scope.row)"
          >详情</el-button>
          <el-button
            v-if="scope.row.orderStatus==201"
            v-permission="['POST /admin/order/ship']"
            type="primary"
            size="mini"
            @click="handleShip(scope.row)"
          >发货</el-button>
          <el-button
            v-if="scope.row.orderStatus==202"
            v-permission="['POST /admin/order/refund']"
            type="primary"
            size="mini"
            @click="handleRefund(scope.row)"
          >退款</el-button>
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

    <!-- 订单详情对话框 -->
    <el-dialog
      :visible.sync="orderDialogVisible"
      title="订单详情"
      width="800"
    >
      <el-form
        :data="orderDetail"
        label-position="left"
        label-width="80px"
      >

        <!-- 订单信息 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >订单信息</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">订单编号：{{ orderDetail.order.orderSn }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">订单状态：<el-tag>{{ orderDetail.order.orderStatus | orderStatusFilter }}</el-tag></el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">下单时间：{{ orderDetail.order.addTime }}</el-col>
          </el-row>
        </el-form-item>

        <!-- 商品信息 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >商品信息</el-col>
          </el-row>
          <el-table
            :data="orderDetail.orderGoods"
            size="small"
            border
            fit
            highlight-current-row
          >
            <el-table-column
              align="center"
              label="商品名称"
              prop="goodsName"
            />
            <el-table-column
              align="center"
              label="商品编号"
              prop="goodsSn"
            />
            <el-table-column
              align="center"
              label="商品规格"
              prop="specifications"
            />
            <el-table-column
              align="center"
              label="商品价格"
              prop="price"
            />
            <el-table-column
              align="center"
              label="商品数量"
              prop="number"
            />
            <el-table-column
              align="center"
              label="商品图片"
              prop="picUrl"
            >
              <template slot-scope="scope">
                <img
                  :src="scope.row.picUrl"
                  width="40"
                >
              </template>
            </el-table-column>
          </el-table>
        </el-form-item>

        <!-- 支付信息 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >支付信息</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">支付渠道：微信支付</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">支付时间：{{ orderDetail.order.payTime }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">支付金额：{{ orderDetail.order.actualPrice }}元</el-col>
          </el-row>
        </el-form-item>

        <!-- 收货信息 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >收货信息</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">收货人：{{ orderDetail.order.consignee }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">收货电话：{{ orderDetail.order.mobile }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">地址：{{ orderDetail.order.address }}</el-col>
          </el-row>
        </el-form-item>

        <!-- 快递信息 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >快递信息</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">快递公司：{{ orderDetail.order.shipChannel }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">快递单号：{{ orderDetail.order.shipSn }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">发货时间：{{ orderDetail.order.shipTime }}</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">确认收货时间：{{ orderDetail.order.confirmTime }}</el-col>
          </el-row>
        </el-form-item>

        <!-- 备注 -->
        <el-form-item>
          <el-row>
            <el-col
              :span="24"
              class="section-title"
            >备注</el-col>
          </el-row>
          <el-row class="content-row">
            <el-col :span="24">{{ orderDetail.order.message }}</el-col>
          </el-row>
        </el-form-item>

      </el-form>
    </el-dialog>
    <!-- 发货对话框 -->
    <el-dialog
      :visible.sync="shipDialogVisible"
      title="发货"
    >
      <el-form
        ref="shipForm"
        :model="shipForm"
        status-icon
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left:50px;"
      >
        <!--        <el-form-item label="快递公司" prop="shipChannel">
          <el-select v-model="shipForm.shipChannel">
            <el-option v-for="item in shipChannelList" :key="item.value" :label="item.label" :value="item.value"/>
          </el-select>
        </el-form-item>-->
        <el-form-item
          label="快递编号"
          prop="shipSn"
        >
          <el-input v-model="shipForm.shipSn" />
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="shipDialogVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click="confirmShip"
        >确定</el-button>
      </div>
    </el-dialog>

    <!-- 退款对话框 -->
    <el-dialog
      :visible.sync="refundDialogVisible"
      title="退款"
    >
      <el-form
        ref="refundForm"
        :model="refundForm"
        status-icon
        label-position="left"
        label-width="100px"
        style="width: 400px; margin-left:50px;"
      >
        <el-form-item
          label="退款金额"
          prop="refundMoney"
        >
          <el-input
            v-model="refundForm.refundMoney"
            :disabled="true"
          />
        </el-form-item>
      </el-form>
      <div
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="refundDialogVisible = false">取消</el-button>
        <el-button
          type="primary"
          @click="confirmRefund"
        >确定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { listOrder, shipOrder, refundOrder, detailOrder, listShipChannel } from '@/api/business/order'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination
import checkPermission from '@/utils/permission' // 权限判断函数

const statusMap = {
  '待付款': [101],
  '已取消': [102, 103],
  '待发货': [201],
  '待收货': [301],
  '退货/售后': [202],
  '已处理': [203],
  '待评价': [401, 402],
  '已完成': [403]
}

export default {
  name: 'Order',
  components: { Pagination },
  filters: {
    orderStatusFilter(status) {
      // 遍历 statusMap，查找包含该状态码的键
      for (const [statusLabel, codes] of Object.entries(statusMap)) {
        if (codes.includes(status)) {
          return statusLabel
        }
      }
      return '未知状态' // 如果找不到对应的状态码，返回“未知状态”
    }
  },
  data() {
    return {
      list: undefined,
      total: 0,
      listLoading: true,
      shipChannelList: [],
      listQuery: {
        page: 1,
        limit: 20,
        id: undefined,
        name: undefined,
        orderStatusArray: [],
        sort: 'add_time',
        order: 'desc',
        payStartDate: '',
        payEndDate: ''
      },
      statusMap: {
        '待付款': [101],
        '已取消': [102, 103],
        '待发货': [201],
        '待收货': [301],
        '待处理': [202],
        '已处理': [203],
        '待评价': [401, 402],
        '已完成': [403]
      },
      computed: {
        selectedStatusLabels() {
        // 将选中的状态代码转换为中文描述
          return this.listQuery.orderStatusArray.map(code => {
            for (const [label, codes] of Object.entries(this.statusMap)) {
              if (codes.includes(code)) {
                return label
              }
            }
            return code // 如果没找到匹配的，直接返回 code
          })
        }
      },
      orderDialogVisible: false,
      orderDetail: {
        order: {},
        user: {},
        orderGoods: []
      },
      shipForm: {
        orderId: undefined,
        shipChannel: undefined,
        shipSn: undefined
      },
      shipDialogVisible: false,
      refundForm: {
        orderId: undefined,
        refundMoney: undefined
      },
      refundDialogVisible: false,
      downloadLoading: false
    }
  },
  created() {
    this.getList()
    this.getListShipChannel()
  },
  methods: {
    checkPermission,
    getList() {
      this.listLoading = true
      listOrder(this.listQuery).then(response => {
        this.list = response.data.data.items
        this.total = response.data.data.total
        this.listLoading = false
      }).catch(() => {
        this.list = []
        this.total = 0
        this.listLoading = false
      })
    },
    getListShipChannel() {
      listShipChannel().then(response => {
        this.shipChannelList = response.data.data.shipChannelList
      })
    },
    handleFilter() {
      // 将选择的订单分类转换为状态数组
      this.listQuery.orderStatusArray = this.listQuery.orderStatusArray.flat()

      // 校验并格式化日期
      const isValidDate = (date) => /^\d{8}$/.test(date)
      this.listQuery.payStartDate = isValidDate(this.listQuery.payStartDate)
        ? this.listQuery.payStartDate
        : this.formatDateToYMD(this.listQuery.payStartDate)
      this.listQuery.payEndDate = isValidDate(this.listQuery.payEndDate)
        ? this.listQuery.payEndDate
        : this.formatDateToYMD(this.listQuery.payEndDate)

      this.listQuery.page = 1
      this.getList()
    },
    handleDetail(row) {
      detailOrder(row.id).then(response => {
        this.orderDetail = response.data.data
      })
      this.orderDialogVisible = true
    },
    handleShip(row) {
      this.shipForm.orderId = row.id
      this.shipForm.shipChannel = row.shipChannel
      this.shipForm.shipSn = row.shipSn

      this.shipDialogVisible = true
      this.$nextTick(() => {
        this.$refs['shipForm'].clearValidate()
      })
    },
    confirmShip() {
      this.$refs['shipForm'].validate((valid) => {
        if (valid) {
          shipOrder(this.shipForm).then(response => {
            this.shipDialogVisible = false
            this.$notify.success({
              title: '成功',
              message: '确认发货成功'
            })
            this.getList()
          }).catch(response => {
            this.$notify.error({
              title: '失败',
              message: response.data.errmsg
            })
          })
        }
      })
    },
    handleRefund(row) {
      this.refundForm.orderId = row.id
      this.refundForm.refundMoney = row.actualPrice

      this.refundDialogVisible = true
      this.$nextTick(() => {
        this.$refs['refundForm'].clearValidate()
      })
    },
    confirmRefund() {
      this.$refs['refundForm'].validate((valid) => {
        if (valid) {
          refundOrder(this.refundForm).then(response => {
            this.refundDialogVisible = false
            this.$notify.success({
              title: '成功',
              message: '确认退款成功'
            })
            this.getList()
          }).catch(response => {
            this.$notify.error({
              title: '失败',
              message: response.data.errmsg
            })
          })
        }
      })
    },
    formatDateToYMD(date) {
      if (!date) return ''
      const d = new Date(date)
      if (isNaN(d.getTime())) return ''

      const year = d.getFullYear().toString()
      const month = (d.getMonth() + 1).toString().padStart(2, '0')
      const day = d.getDate().toString().padStart(2, '0')
      return `${year}${month}${day}`
    },

    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['订单ID', '订单编号', '用户ID', '订单状态', '是否删除', '收货人', '收货联系电话', '收货地址']
        const filterVal = ['id', 'orderSn', 'userId', 'orderStatus', 'isDelete', 'consignee', 'mobile', 'address']
        excel.export_json_to_excel2(tHeader, this.list, filterVal, '订单信息')
        this.downloadLoading = false
      })
    },
    resetFilters() {
      // 重置查询条件
      this.listQuery.userId = ''
      this.listQuery.orderSn = ''
      this.listQuery.orderStatusArray = []
      this.listQuery.payStartDate = ''
      this.listQuery.payEndDate = ''
      this.listQuery.page = 1

      // 重新获取列表数据
      this.getList()
    }
  }
}
</script>
<style scoped>
/* Main dialog title styling */
.el-dialog__title {
  font-size: 20px;
  font-weight: bold;
}

/* Section titles (小标题) */
.section-title {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-top: 15px;
  margin-bottom: 5px;
}

/* Content rows (正文) */
.content-row {
  padding-left: 15px;
  font-size: 14px;
  color: #666;
  line-height: 1.8; /* Adjusts spacing between lines */
}

.el-form-item {
  margin-bottom: 12px;
}
</style>
