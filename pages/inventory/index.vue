<template>
  <view class="inventory-page">
    <!-- 搜索区域 -->
    <view class="search-area">
      <uni-forms ref="searchForm" :model="searchForm">
        <uni-row :gutter="10">
          <uni-col :span="8">
            <uni-forms-item label="存货编码">
              <uni-easyinput v-model="searchForm.code" placeholder="请输入存货编码" />
            </uni-forms-item>
          </uni-col>
          <uni-col :span="8">
            <uni-forms-item label="存货名称">
              <uni-easyinput v-model="searchForm.name" placeholder="请输入存货名称" />
            </uni-forms-item>
          </uni-col>
          <uni-col :span="8">
            <uni-forms-item label="助记码">
              <uni-easyinput v-model="searchForm.mnemonicCode" placeholder="请输入助记码" />
            </uni-forms-item>
          </uni-col>
        </uni-row>
        <uni-row class="button-group">
          <button type="primary" @click="handleSearch">查询</button>
          <button type="default" @click="handleReset">重置</button>
        </uni-row>
      </uni-forms>
    </view>

    <!-- 数据列表 -->
    <view class="table-area">
      <uni-table ref="table" :loading="loading" border stripe emptyText="暂无数据">
        <uni-tr>
          <uni-th align="center">存货编码</uni-th>
          <uni-th align="center">存货名称</uni-th>
          <uni-th align="center">规格型号</uni-th>
          <uni-th align="center">计量单位</uni-th>
          <uni-th align="center">分类</uni-th>
          <uni-th align="center">现存量</uni-th>
          <uni-th align="center">可用量</uni-th>
        </uni-tr>
        <uni-tr v-for="(item, index) in tableData" :key="index">
          <uni-td>{{ item.code }}</uni-td>
          <uni-td>{{ item.name }}</uni-td>
          <uni-td>{{ item.specification }}</uni-td>
          <uni-td>{{ item.unitName }}</uni-td>
          <uni-td>{{ item.className }}</uni-td>
          <uni-td>{{ item.currentStock }}</uni-td>
          <uni-td>{{ item.availableStock }}</uni-td>
        </uni-tr>
      </uni-table>
      
      <!-- 分页器 -->
      <view class="pagination">
        <uni-pagination
          :total="total"
          :pageSize="pageSize"
          :current="current"
          @change="handlePageChange"
        />
      </view>
    </view>
  </view>
</template>

<script>
const API_BASE_URL = 'http://222.174.223.138:3301/client/'

export default {
  data() {
    return {
      searchForm: {
        code: '',
        name: '',
        mnemonicCode: ''
      },
      loading: false,
      tableData: [],
      total: 0,
      current: 1,
      pageSize: 10
    }
  },
  onLoad() {
    this.loadData()
  },
  methods: {
    // 构建查询SQL
    buildQuerySQL() {
      let sql = `
        SELECT TOP (100) PERCENT 
          dbo.AA_InventoryEntity.code AS 编码,
          dbo.AA_InventoryEntity.name AS 名称,
          dbo.AA_InventoryEntity.specification AS 规格,
          dbo.AA_Unit.name AS 计量单位,
          dbo.AA_InventoryClass.name AS 分类,
          dbo.V_ST_CurrentStockWithZero.BaseQuantity AS 现存量,
          dbo.V_ST_CurrentStockWithZero.CanuseBaseQuantity AS 可用量
        FROM dbo.AA_InventoryEntity
        INNER JOIN dbo.AA_Unit ON dbo.AA_InventoryEntity.idunit = dbo.AA_Unit.id
        INNER JOIN dbo.AA_InventoryClass ON dbo.AA_InventoryEntity.idinventoryclass = dbo.AA_InventoryClass.id
        INNER JOIN dbo.V_ST_CurrentStockWithZero ON dbo.AA_InventoryEntity.id = dbo.V_ST_CurrentStockWithZero.idinventory
        WHERE 1=1
      `
      
      if (this.searchForm.code) {
        sql += ` AND dbo.AA_InventoryEntity.code LIKE '%${this.searchForm.code}%'`
      }
      if (this.searchForm.name) {
        sql += ` AND dbo.AA_InventoryEntity.name LIKE '%${this.searchForm.name}%'`
      }
      if (this.searchForm.mnemonicCode) {
        sql += ` AND dbo.AA_InventoryEntity.mnemonicCode LIKE '%${this.searchForm.mnemonicCode}%'`
      }
      
      return sql
    },
    
    // 加载数据
    async loadData() {
      this.loading = true
      try {
        const params = {
          sql: this.buildQuerySQL(),
          pagesize: this.pageSize,
          pageno: this.current
        }
        
        const response = await uni.request({
          url: API_BASE_URL + 'sql/getRows',
          method: 'POST',
          data: params
        })
        
        if (response.data.success) {
          this.tableData = response.data.rows.map(item => ({
            code: item.编码,
            name: item.名称,
            specification: item.规格,
            unitName: item.计量单位,
            className: item.分类,
            currentStock: item.现存量,
            availableStock: item.可用量
          }))
          this.total = response.data.cnt
        }
      } catch (error) {
        uni.showToast({
          title: '加载数据失败',
          icon: 'none'
        })
      } finally {
        this.loading = false
      }
    },
    
    // 处理查询
    handleSearch() {
      this.current = 1
      this.loadData()
    },
    
    // 处理重置
    handleReset() {
      this.searchForm = {
        code: '',
        name: '',
        mnemonicCode: ''
      }
      this.handleSearch()
    },
    
    // 处理分页变化
    handlePageChange(e) {
      this.current = e.current
      this.loadData()
    }
  }
}
</script>

<style lang="scss">
.inventory-page {
  padding: 20rpx;
  
  .search-area {
    background-color: #fff;
    padding: 20rpx;
    border-radius: 8rpx;
    margin-bottom: 20rpx;
    
    .button-group {
      margin-top: 20rpx;
      display: flex;
      justify-content: center;
      gap: 20rpx;
    }
  }
  
  .table-area {
    background-color: #fff;
    padding: 20rpx;
    border-radius: 8rpx;
    
    .pagination {
      margin-top: 20rpx;
      display: flex;
      justify-content: center;
    }
  }
}
</style> 