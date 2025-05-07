<template>
  <view class="sale-order-page">
    <uni-forms ref="form" :model="formData" :rules="rules">
      <uni-forms-item label="客户" required name="customer">
        <uni-easyinput
          v-model="customerSearch"
          placeholder="请输入客户名称"
          @input="(val) => handleCustomerSearch(val)"
        />
        <view class="search-results" v-if="showCustomerResults">
          <view 
            class="result-item" 
            v-for="item in customerResults" 
            :key="item.Code"
            @click="selectCustomer(item)"
          >
            {{ item.Name }}
          </view>
        </view>
      </uni-forms-item>

      <uni-forms-item label="商品明细" required name="details">
        <view class="detail-list">
          <view class="detail-item" v-for="(item, index) in formData.SaleOrderDetails" :key="index">
            <view class="detail-header">
              <text>商品{{index + 1}}</text>
              <button type="warn" size="mini" @click="removeDetail(index)">删除</button>
            </view>
            
            <uni-forms-item label="商品" required>
              <uni-easyinput
                v-model="item.inventorySearch"
                placeholder="请输入商品名称"
                @input="(val) => handleInventorySearch(val, index)"
              />
              <view class="search-results" v-if="item.showResults">
                <view 
                  class="result-item" 
                  v-for="inv in item.inventoryResults" 
                  :key="inv.Code"
                  @click="selectInventory(inv, index)"
                >
                  {{ inv.Name }}
                </view>
              </view>
            </uni-forms-item>

            <uni-forms-item label="数量" required>
              <uni-easyinput
                v-model="item.Quantity"
                type="number"
                placeholder="请输入数量"
              />
            </uni-forms-item>

            <uni-forms-item label="单价" required>
              <uni-easyinput
                v-model="item.OrigTaxPrice"
                type="number"
                placeholder="请输入单价"
              />
            </uni-forms-item>
          </view>
        </view>
        <button type="primary" @click="addDetail">添加商品</button>
      </uni-forms-item>

      <button type="primary" @click="submitForm">提交订单</button>
    </uni-forms>
  </view>
</template>

<script>
const API_BASE_URL = 'http://cjt.yuanding.tech/tp'

export default {
  data() {
    return {
      customerSearch: '',
      customerResults: [],
      showCustomerResults: false,
      formData: {
        ExternalCode: '',
        Customer: {
          Code: ''
        },
        DynamicPropertyKeys: ['isautoaudit'],
        DynamicPropertyValues: [true],
        SaleOrderDetails: []
      },
      rules: {
        customer: {
          rules: [{
            required: true,
            errorMessage: '请选择客户'
          }]
        },
      }
    }
  },
  methods: {
    // 客户搜索
    async handleCustomerSearch(value) {
      if (!value) {
        this.showCustomerResults = false
        return
      }
      try {
        const response = await uni.request({
          url: `${API_BASE_URL}/tplus/api/v2/partner/Query`,
          method: 'POST',
          data: {
            params: {
              Name: value
            }
          }
        })
        if (response.data) {
          this.customerResults = response.data
          this.showCustomerResults = true
        }
      } catch (error) {
        uni.showToast({
          title: '获取客户信息失败',
          icon: 'none'
        })
      }
    },

    // 选择客户
    selectCustomer(customer) {
      this.formData.Customer.Code = customer.Code
      this.customerSearch = customer.Name
      this.showCustomerResults = false
    },

    // 商品搜索
    async handleInventorySearch(value, index) {
      if (!value) {
        this.formData.SaleOrderDetails[index].showResults = false
        return
      }
      try {
        const response = await uni.request({
          url: `${API_BASE_URL}/tplus/api/v2/inventory/Query`,
          method: 'POST',
          data: {
            params: {
              Name: value
            }
          }
        })
        if (response.data) {
          this.formData.SaleOrderDetails[index].inventoryResults = response.data
          this.formData.SaleOrderDetails[index].showResults = true
        }
      } catch (error) {
        uni.showToast({
          title: '获取商品信息失败',
          icon: 'none'
        })
      }
    },

    // 选择商品
    selectInventory(inventory, index) {
      const detail = this.formData.SaleOrderDetails[index]
      detail.Inventory = {
        Code: inventory.Code
      }
      detail.inventorySearch = inventory.Name
      detail.showResults = false
    },

    // 添加商品明细
    addDetail() {
      this.formData.SaleOrderDetails.push({
        inventorySearch: '',
        inventoryResults: [],
        showResults: false,
        Quantity: '',
        OrigTaxPrice: '',
        Unit: {
          Name: '个'
        },
        DynamicPropertyKeys: ['priuserdefnvc1', 'priuserdefdecm1'],
        DynamicPropertyValues: ['', '']
      })
    },

    // 删除商品明细
    removeDetail(index) {
      this.formData.SaleOrderDetails.splice(index, 1)
    },

    // 提交表单
    async submitForm() {
      if(this.formData.SaleOrderDetails.length==0){
        uni.showToast({
          title: '请添加商品明细',
          icon: 'none'
        })
        return
      }
      try {
        const valid = await this.$refs.form.validate()
        if (!valid) return

        const response = await uni.request({
          url: `${API_BASE_URL}/tplus/api/v2/saleOrder/Create`,
          method: 'POST',
          data: {
            dto: this.formData
          }
        })

        if (response.data) {
          uni.showToast({
            title: '订单创建成功',
            icon: 'success'
          })
          // 重置表单
          this.formData = {
            ExternalCode: '',
            Customer: {
              Code: ''
            },
            DynamicPropertyKeys: ['isautoaudit'],
            DynamicPropertyValues: [true],
            SaleOrderDetails: []
          }
          this.customerSearch = ''
        }
      } catch (error) {
        console.log(error)
      }
    }
  }
}
</script>

<style lang="scss">
.sale-order-page {
  padding: 20rpx;
  height: 100vh;
  overflow-y: auto;

  :deep(.uni-forms-item__label) {
    min-width: 80rpx !important;
  }

  .search-results {
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: #fff;
    border: 1px solid #eee;
    border-radius: 4rpx;
    max-height: 300rpx;
    overflow-y: auto;
    z-index: 100;

    .result-item {
      padding: 20rpx;
      border-bottom: 1px solid #eee;

      &:last-child {
        border-bottom: none;
      }

      &:active {
        background-color: #f5f5f5;
      }
    }
  }

  .detail-list {
    .detail-item {
      background: #f8f8f8;
      padding: 20rpx;
      margin-bottom: 20rpx;
      border-radius: 8rpx;

      .detail-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 20rpx;
      }
    }
  }
}
</style> 