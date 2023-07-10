<template>
  <el-card class="category-container">
    <template #header>
      <div class="header">
        <el-popconfirm
            title="确定删除吗？"
            @confirm="handleDelete"
        >
          <template #reference>
            <el-button type="danger" size="small" icon="el-icon-delete">删除</el-button>
          </template>
        </el-popconfirm>
        <el-popconfirm
            title="确定配货吗？"
            @confirm="handleCheckDone"
        >
          <template #reference>
            <el-button type="primary" size="small" icon="el-icon-place">配货</el-button>
          </template>
        </el-popconfirm>
        <el-popconfirm
            title="确定出库吗？"
            @confirm="handleCheckOut"
        >
          <template #reference>
            <el-button type="primary" size="small" icon="el-icon-check">出库</el-button>
          </template>
        </el-popconfirm>
        <el-popconfirm
            title="确定关闭吗？"
            @confirm="handleCheckClose"
        >
          <template #reference>
            <el-button type="danger" size="small" icon="el-icon-error">关闭订单</el-button>
          </template>
        </el-popconfirm>
        <el-popconfirm
            title="确认完成吗？"
            @confirm="handleCheckFinish"
        >
          <template #reference>
            <el-button  type="primary" size="small" icon="el-icon-check">确认完成</el-button>
          </template>
        </el-popconfirm>
      </div>
    </template>
    <el-table
        border
        v-slot="{row}"
        v-loading="loading"
        ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100% "
        @selection-change="handleSelectionChange">
      <el-table-column
          type="selection"
          width="55"
      >
      </el-table-column>
      <el-table-column
          prop="orderId"
          label="ID"
          width="50">
      </el-table-column>
      <el-table-column
          prop="orderNo"
          label="订单号"
          width="200">
      </el-table-column>
      <el-table-column
          prop="userId"
          label="用户 ID"
          width="80">
      </el-table-column>
      <el-table-column
          prop="totalPrice"
          label="订单总额"
          width="100">
      </el-table-column>
      <el-table-column
          prop="payType"
          label="支付方式"
          width="100">
        <template v-slot="{row}">
          <span v-if="row.payType===0">无</span>
          <span v-else-if="row.payType===1">支付宝支付</span>
          <span v-else-if="row.payType===2">微信支付</span>
          <span v-else>未知类型</span>
        </template>
      </el-table-column>
      <el-table-column
          prop="payStatus"
          label="支付状态"
          width="100">
        <template v-slot="{row}">
          <span v-if="row.payStatus===0">未支付</span>
          <span v-else-if="row.payStatus===1">支付成功</span>
          <span v-else-if="row.payStatus===-1">支付失败</span>
          <span v-else>未知状态</span>
        </template>
      </el-table-column>
      <el-table-column
          prop="payTime"
          label="支付时间"
          width="150">
      </el-table-column>
      <el-table-column
          prop="orderStatus"
          label="订单状态"
          width="100">
        <template v-slot="{ row }">
          <span v-if="row.orderStatus===0">待支付</span>
          <span v-else-if="row.orderStatus===1">已支付</span>
          <span v-else-if="row.orderStatus===2">配货完成</span>
          <span v-else-if="row.orderStatus===3">出库成功</span>
          <span v-else-if="row.orderStatus===4">交易成功</span>
          <span v-else-if="row.orderStatus===-1">手动关闭</span>
          <span v-else-if="row.orderStatus===-2">超时关闭</span>
          <span v-else-if="row.orderStatus===-3">商家关闭</span>
          <span v-else>未知状态</span>
        </template>
      </el-table-column>
      <el-table-column
          label="其他信息"
          width="110">
        <template v-slot="{row}">
          <el-button type="info" round @click="handleEdit(row.orderId)">查看详情</el-button>
        </template>
      </el-table-column>
      <el-table-column
          prop="createTime"
          label="创建时间"
          width="150">
      </el-table-column>
      <el-table-column
          prop="updateTime"
          label="更新时间"
          width="150">
      </el-table-column>
    </el-table>


    <!--    总数超过一页，再展示分页器-->
    <el-pagination
        background
        layout="prev, pager, next"
        :total="total"
        :page-size="pageSize"
        :current-page="currentPage"
        @current-change="changePage"
    />
  </el-card>
<!--  <OrderDetail :order-id="currOrderId"/>-->
</template>

<script>
import {onMounted, reactive, ref, toRefs, computed} from 'vue'
import { ElMessage } from 'element-plus'
import axios from "../utils/axios";
import router from "../router";
// import OrderDetail from '../views/OrderDetail'


export default {
  name: 'Order',
  components:{
    // OrderDetail
  },

  setup() {
    const currOrderId = ref(null);
    const state = reactive({
      loading: false,
      tableData: [], // 数据列表
      total: 0, // 总条数
      currentPage: 1, // 当前页
      pageSize: 10, // 分页大小
      orderNo: "",
      orderStatus:null,
    })
    onMounted(() => {
      getOrderList()
    })
    // 选择项
    const handleSelectionChange = (val) => {
      state.multipleSelection = val
    }
    const search = ref('')
    const searchData = computed(() =>
        state.tableData.filter(
            (data) =>
                !search.value ||
                data.name.toLowerCase().includes(search.value.toLowerCase())
        )
    )

    //查看详情
    const handleEdit=(orderId)=>{
      currOrderId.value=orderId
      console.log("orderId:"+orderId)
      router.push({path:'/order_detail',query: {orderId:orderId}})
    }

    // 批量删除
    const handleDelete = () => {
      if (!state.multipleSelection.length) {
        ElMessage.error('请选择项')
        return
      }
      axios.put('/orders/deleteOrder', {
          ids: state.multipleSelection.map(i => i.orderId)
      }).then(() => {
        ElMessage.success('删除成功')
        getOrderList()
      })
    }

    // 批量配货
    const handleCheckDone = () => {
      if (!state.multipleSelection.length) {
        ElMessage.error('请选择项')
        return
      }
//向后端 /order/checkDown 发送一个PUT请求，请求体包括一个对象data，其中一个属性是
      //ids,是从state的multipleSelection属性中取出的记录中的 orderId属性所组成的数组
      axios.put('/orders/checkDone', {
        ids: state.multipleSelection.map(i => i.orderId)
      }).then(() => {
        ElMessage.success('配货成功')
        getOrderList()
      })
    }

    // 批量出库
    const handleCheckOut = () => {
      if (!state.multipleSelection.length) {
        ElMessage.error('请选择项')
        return
      }
//向后端 /order/checkDown 发送一个PUT请求，请求体包括一个对象data，其中一个属性是
      //ids,是从state的multipleSelection属性中取出的记录中的 orderId属性所组成的数组
      axios.put('/orders/checkOut', {
        ids: state.multipleSelection.map(i => i.orderId)
      }).then(() => {
        ElMessage.success('出库成功')
        getOrderList()
      })
    }

    // 批量关闭
    const handleCheckClose = () => {
      if (!state.multipleSelection.length) {
        ElMessage.error('请选择项')
        return
      }
//向后端 /order/checkDown 发送一个PUT请求，请求体包括一个对象data，其中一个属性是
      //ids,是从state的multipleSelection属性中取出的记录中的 orderId属性所组成的数组
      axios.put('/orders/close', {
        ids: state.multipleSelection.map(i => i.orderId)
      }).then(() => {
        ElMessage.success('关闭成功')
        getOrderList()
      })
    }

    // 批量完成
    const handleCheckFinish = () => {
      if (!state.multipleSelection.length) {
        ElMessage.error('请选择项')
        return
      }
//向后端 /order/checkDown 发送一个PUT请求，请求体包括一个对象data，其中一个属性是
      //ids,是从state的multipleSelection属性中取出的记录中的 orderId属性所组成的数组
      axios.put('/orders/checkFinish', {
        ids: state.multipleSelection.map(i => i.orderId)
      }).then(() => {
        ElMessage.success('确认完成')
        getOrderList()
      })
    }

    // 获取后端订单列表
    const getOrderList = () => {
      state.loading = true
      axios.get('/orders', {
        //后端的參數
        /**
         *         params.put("page", pageNumber);
         *         params.put("limit", pageSize);
         *         params.put("orderNo", orderNo);
         *         params.put("orderStatus", orderStatus);
         */
        params: {
          pageNumber: state.currentPage,
          pageSize: state.pageSize,
          orderStatus:state.orderStatus,
          orderNo:state.orderNo
        }
      }).then(res => {
        state.tableData = res.list
        state.total = res.totalCount
        state.currentPage = res.currPage
        state.loading = false
      })
    }

    // watch(state.tableData, filteredTableData);



    const changePage = (val) => {
      state.currentPage = val
      getOrderList()
    }

    // const filteredTableData=computed(() => {
    //   // state.tableData = this.tableData.values.filter(record => record.isDeleted === 0);
    //   return state.tableData.values.filter(record => record.isDeleted === 0);
    //   // state.tableData = this.tableData.map()
    // });

    return {
      ...toRefs(state),
      getOrderList,
      changePage,
      searchData,
      search,
      handleDelete,
      handleSelectionChange,
      handleCheckDone,
      handleCheckClose,
      handleCheckOut,
      handleCheckFinish,
      handleEdit,
      // currOrderId
      // handleNext
    }
  }
}
</script>


<style scoped>
.category-container {
  min-height: 100%;
}
.el-card.is-always-shadow {
  min-height: 100%!important;
}
.el-link {
  margin-right: 8px;
}
.el-link .el-icon--right.el-icon {
  vertical-align: text-bottom;
}
</style>