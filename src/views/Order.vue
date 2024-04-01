<template>
  <el-card class="order-container">
    <template #header>
      <div class="header">
        <el-input
          style="width: 200px; margin-right: 10px"
          placeholder="请输入订单号"
          v-model="state.orderNo"
          @change="handleOption"
          clearable
        />
        <el-select @change="handleOption" v-model="state.order_status" style="width: 200px; margin-right: 10px">
          <el-option
            v-for="item in state.options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
        <!-- <el-button type="primary" size="small" icon="el-icon-edit">修改订单</el-button> -->
        <el-button type="primary" :icon="HomeFilled" @click="handleConfig()">接单</el-button>
        <el-button type="primary" :icon="HomeFilled" @click="handleSend()">出餐</el-button>
        <el-button type="danger" :icon="Delete" @click="handleClose()">关闭订单</el-button>
      </div>
    </template>
    <el-table
      :load="state.loading"
      :data="state.tableData"
      tooltip-effect="dark"
      style="width: 100%"
      @selection-change="handleSelectionChange">
      <el-table-column
        type="selection"
        width="55">
      </el-table-column>
      <el-table-column
        prop="order_id"
        label="订单号"
      >
      </el-table-column>
      <el-table-column
        prop="goods_price"
        label="商品单价"
      >
      </el-table-column> <el-table-column
        prop="goods_num"
        label="订单数量"
      >
      </el-table-column>
      <el-table-column
        prop="order_status"
        label="订单状态"
      >
        <template #default="scope">
          <span>{{ $filters.orderMap(scope.row.order_status) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="payway"
        label="支付方式"
      >
        <template #default='scope'>
          <span v-if="scope.row.payway == 1">微信支付</span>
          <span v-else-if="scope.row.payway== 2">支付宝支付</span>
          <span v-else>未知</span>
        </template>
      </el-table-column>
      <!-- <el-table-column
        prop="createTime"
        label="创建时间"
      >
      </el-table-column> -->
      <el-table-column
        label="操作"
      >
        <template #default="scope">
          <el-popconfirm
            v-if="scope.row.order_status == 3"
            title="确定接单完成吗？"
            @confirm="handleConfig(scope.row.order_id)"
            confirm-button-text="确定"
            cancel-button-text="取消"
          >
            <template #reference>
              <a style="cursor: pointer; margin-right: 10px">接单</a>
            </template>
          </el-popconfirm>
          <el-popconfirm
            v-if="scope.row.order_status == 4"
            title="确定出餐吗？"
            @confirm="handleSend(scope.row.order_id)"
            confirm-button-text="确定"
            cancel-button-text="取消"
          >
            <template #reference>
              <a style="cursor: pointer; margin-right: 10px">出餐</a>
            </template>
          </el-popconfirm>
          <el-popconfirm
            v-if="!(scope.row.order_status == 6 || scope.row.order_status < 0)"
            title="确定关闭订单吗？"
            @confirm="handleClose(scope.row.order_id)"
            confirm-button-text="确定"
            cancel-button-text="取消"
          >
            <template #reference>
              <a style="cursor: pointer; margin-right: 10px">关闭订单</a>
            </template>
          </el-popconfirm>
          <router-link :to="{ path: '/order_detail', query: { id: scope.row.order_id }}">订单详情</router-link>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background
      layout="prev, pager, next"
      :total="state.total"
      :page-size="state.pageSize"
      :current-page="state.currentPage"
      @current-change="changePage"
    />
  </el-card>
</template>

<script setup>
import { onMounted, reactive } from 'vue'
import { ElMessage } from 'element-plus'
import { HomeFilled, Delete } from '@element-plus/icons-vue'
import axios from '@/utils/axios'

const state = reactive({
  loading: false,
  tableData: [], // 数据列表
  multipleSelection: [], // 选中项
  total: 0, // 总条数
  currentPage: 1, // 当前页
  pageSize: 10, // 分页大小
  orderNo: '', // 订单号
  order_status: '', // 订单状态
  // 订单状态筛选项默认值
  options: [{
    value: '',
    label: '全部'
  }, {
    value: 2,
    label: '待支付'
  }, {
    value: 3,
    label: '已支付'
  }, {
    value: 4,
    label: '接单完成'
  }, {
    value: 5,
    label: '出餐成功'
  }, {
    value: 7,
    label: '交易成功'
  }, {
    value: -1,
    label: '手动关闭'
  }, {
    value: 0,
    label: '超时关闭'
  }, {
    value: -3,
    label: '商家关闭'
  }]
})
// 初始化获取订单列表
onMounted(() => {
  getOrderList()
})
// 获取列表方法
const getOrderList = () => {
  state.loading = true
  axios.get('/foo/order/list', {
    params: {
      pageNumber: state.currentPage,
      pageSize: state.pageSize,
      orderNo: state.orderNo,
      order_status: state.order_status
    }
  }).then(res => {
    console.log(res);
    state.tableData = res.list
    state.total = res.totalCount
    state.currentPage = res.currPage
    state.loading = false
  })
}
// 触发过滤项方法
const handleOption = () => {
  state.currentPage = 1
  getOrderList()
}
// checkbox 选择项
const handleSelectionChange = (val) => {
  state.multipleSelection = val
}
// 翻页方法
const changePage = (val) => {
  state.currentPage = val
  getOrderList()
}
// 接单方法
const handleConfig = (id) => {
  let params
  // 当个配置
  if (id) {
    params = [id]
  } else {
    if (!state.multipleSelection.length) {
      console.log('state.multipleSelection', state.multipleSelection.length)
      ElMessage.error('请选择项')
      return
    }
    // 多选配置
    params = state.multipleSelection.map(i => i.order_id)
  }
  axios.put('/foo/orders/checkDone', {
    ids: params
  }).then(() => {
    ElMessage.success('接单成功')
    getOrderList()
  })
}
// 出餐方法
const handleSend = (id) => {
  let params
  if (id) {
    params = [id]
  } else {
    if (!state.multipleSelection.length) {
      ElMessage.error('请选择项')
      return
    }
    params = state.multipleSelection.map(i => i.order_id)
  }
  axios.put('/orders/checkOut', {
    ids: params
  }).then(() => {
    ElMessage.success('出餐成功')
    getOrderList()
  })
}
// 关闭订单方法
const handleClose = (id) => {
  let params
  if (id) {
    params = [id]
  } else {
    if (!state.multipleSelection.length) {
      ElMessage.error('请选择项')
      return
    }
    params = state.multipleSelection.map(i => i.order_id)
  }
  axios.put('/orders/close', {
    ids: params
  }).then(() => {
    ElMessage.success('关闭成功')
    getOrderList()
  })
}
</script>