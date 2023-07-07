<template>
  <el-card class="category-container">
    i am order!!!!!!!

  <el-table
      v-loading="loading"
      :data="tableData"
      tooltip-effect="dark"
      style="width: 100%">
    <el-table-column
        prop="orderId"
        label="订单 ID"
        width="200">
    </el-table-column>
    <el-table-column
        prop="orderNo"
        label="订单号"
        width="200">
    </el-table-column>
    <el-table-column
        prop="userId"
        label="用户 ID"
        width="200">
    </el-table-column>
    <el-table-column
        prop="totalPrice"
        label="订单总额"
        width="200">
    </el-table-column>
    <el-table-column
        prop="payStatus"
        label="支付状态"
        width="200">
    </el-table-column>
    <el-table-column
        prop="payTime"
        label="支付时间"
        width="200">
    </el-table-column>
    <el-table-column
        prop="orderStatus"
        label="订单状态"
        width="200">
    </el-table-column>
    <el-table-column
        prop="extraInfo"
        label="其他信息"
        width="200">
    </el-table-column>
  </el-table>

    <!--总数超过一页，再展示分页器-->
<!--    <el-pagination-->
<!--        background-->
<!--        layout="prev, pager, next"-->
<!--        :total="total"-->
<!--        :page-size="pageSize"-->
<!--        :current-page="currentPage"-->
<!--        @current-change="changePage"-->
<!--    />-->
  </el-card>
</template>

<script>
import { onMounted, onUnmounted, reactive, ref, toRefs,computed  } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import DialogAddCategory from '@/components/DialogAddCategory.vue'
// import axios from '@/utils/axios'
import axios from "../utils/axios";

export default {
  name: 'Order',
  setup() {
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

    const changePage = (val) => {
      state.currentPage = val
      getOrderList()
    }
    const filteredTableData = computed(() => {
      state.tableData = this.tableData.values.filter(record => record.isDeleted !== 1);
    });

    return {
      ...toRefs(state),
      getOrderList,
      changePage,
      filteredTableData,
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
</style>