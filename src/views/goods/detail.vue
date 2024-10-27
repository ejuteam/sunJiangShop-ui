<template>
  <div class="app-container">
    <!-- 商品介绍板块 -->
    <el-card class="section-card">
      <div slot="header" class="header-title">商品介绍</div>
      <el-table :data="introductionData" border>
        <el-table-column property="label" label="字段" width="200" />
        <el-table-column property="value" label="内容" />
      </el-table>
    </el-card>

    <!-- 商品规格板块 -->
    <el-card class="section-card">
      <div slot="header" class="header-title">商品规格</div>
      <el-table :data="specifications" border>
        <el-table-column property="specification" label="规格名称" width="200" />
        <el-table-column property="weight" label="重量(g)" width="120" />
        <el-table-column property="repertory" label="库存(件)" width="120" />
        <el-table-column property="price" label="售价(元)" width="120" />
        <el-table-column property="picUrl" label="规格图片">
          <template #default="{ row }">
            <img v-if="row.picUrl" :src="row.picUrl" class="spec-image" />
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 邮费信息板块 -->
    <el-card class="section-card">
      <div slot="header" class="header-title">邮费信息</div>
      <el-table :data="shippingData" border>
        <el-table-column property="label" label="字段" width="200" />
        <el-table-column property="value" label="内容" />
      </el-table>
    </el-card>

    <!-- 操作按钮 -->
    <div class="button-container">
      <el-button @click="goBack">返回</el-button>
      <el-button type="primary" @click="goToEdit">编辑商品</el-button>
    </div>
  </div>
</template>

<script>
import { detailGoods } from '@/api/business/goods'

export default {
  name: 'GoodsDetail',
  data() {
    return {
      goods: {
        gallery: [],
        categoryName: ''
      },
      specifications: [],
      introductionData: [],
      shippingData: []
    }
  },
  created() {
    this.loadGoodsDetail()
  },
  methods: {
    loadGoodsDetail() {
      const goodsId = this.$route.query.id
      if (!goodsId) return

      detailGoods(goodsId).then(response => {
        this.goods = response.data.data.goods
        this.specifications = response.data.data.specifications || []
        this.goods.categoryName = response.data.data.categoryName

        // 商品介绍数据
        this.introductionData = [
          { label: '商品编号', value: this.goods.goodsSn },
          { label: '商品名称', value: this.goods.name },
          { label: '是否为生鲜产品', value: this.goods.isFresh ? '是' : '否' },
          { label: '商品简介', value: this.goods.brief },
          { label: '所属分类', value: this.goods.categoryName },
          { label: '是否上架', value: this.goods.isOnSale ? '上架' : '下架' }
        ]

        // 邮费数据
        this.shippingData = [
          { label: '是否可包邮', value: this.goods.isPostage ? '是' : '否' },
          { label: '包邮下限', value: `${this.goods.minPostage} 元` },
          { label: '首重-重量 (kg)', value: `${this.goods.firstWeight} kg` },
          { label: '首重-邮费 (元)', value: `${this.goods.postageFir} 元` },
          { label: '续重-每千克邮费 (元)', value: `${this.goods.postageSec} 元` }
        ]
      })
    },
    goBack() {
      this.$router.push({ path: '/goods/list' })
    },
    goToEdit() {
      this.$router.push({ path: `/goods/edit?id=${this.$route.query.id}` })
    }
  }
}
</script>

<style scoped>
.app-container {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.section-card {
  width: 100%;
}

.header-title {
  font-weight: bold;
  font-size: 16px;
}

.spec-image {
  width: 40px;
  height: 40px;
  border-radius: 5px;
}

.button-container {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}
</style>
