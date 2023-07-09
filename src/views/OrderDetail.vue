<!--<template>-->
<!--  <el-card class="category-container">-->
<!--    <el-descriptions-->
<!--        :size="size"-->
<!--        class="margin-top"-->
<!--        title="订单详情"-->
<!--        :column="3"-->
<!--        border-->
<!--    >-->
<!--      <el-descriptions-item>-->
<!--        <template #label>-->
<!--          <div class="cell-item">-->
<!--            订单号-->
<!--          </div>-->
<!--        </template>-->
<!--        {{orderNo}}-->
<!--      </el-descriptions-item>-->
<!--      <el-descriptions-item>-->
<!--        <template #label>-->
<!--          <div class="cell-item">-->
<!--            订单总额-->
<!--          </div>-->
<!--        </template>-->
<!--        {{totalPrice}}-->
<!--      </el-descriptions-item>-->
<!--      <el-descriptions-item>-->
<!--        <template #label>-->
<!--          <div class="cell-item">-->
<!--            订单状态-->
<!--          </div>-->
<!--        </template>-->
<!--        {{ orderStatusString.value}}-->
<!--      </el-descriptions-item>-->
<!--      <el-descriptions-item>-->
<!--        <template #label>-->
<!--          <div class="cell-item">-->
<!--            地址-->
<!--          </div>-->
<!--        </template>-->
<!--        {{orderAddress.detailAddress}}-->
<!--      </el-descriptions-item>-->
<!--    </el-descriptions>-->

<!--    <el-divider></el-divider>-->

<!--    <el-table  :data="itemList.values()" :height="200" border>-->
<!--      <el-table-column prop="goodsId" label="商品ID"></el-table-column>-->
<!--      <el-table-column prop="goodsCount" label="商品数量"></el-table-column>-->
<!--      <el-table-column prop="goodsName" label="商品名称"></el-table-column>-->
<!--      <el-table-column prop="sellingPrice" label="售价"></el-table-column>-->
<!--    </el-table>-->
<!--  </el-card>-->
<!--</template>-->

<template>
  <div>
    <el-form
        label-position="left"
        label-width="100px"
        style="max-width: 460px"
    >
      <el-form-item label="订单号">
        <el-input v-model="orderNo" disabled/>
      </el-form-item>
      <el-form-item label="订单总额">
        <el-input v-model="totalPrice" disabled/>
      </el-form-item>
      <el-form-item label="支付方式">
        <el-input v-model="payTypeString" disabled />
      </el-form-item>
      <el-form-item label="订单状态">
        <el-input v-model="orderStatusString" disabled />
      </el-form-item>
      <el-form-item label="用户姓名">
        <el-input v-model="orderAddressInfo.userName"  disabled/>
      </el-form-item>
      <el-form-item label="地址">
        <el-input v-model="orderAddressInfo.detailAddress" disabled/>
      </el-form-item>
    </el-form>

    <el-table :data="itemList" v-loading="loading" :height="200" border>
      <el-table-column prop="goodsId" label="商品ID"></el-table-column>
      <el-table-column prop="goodsCount" label="商品数量"></el-table-column>
      <el-table-column prop="goodsName" label="商品名称"></el-table-column>
      <el-table-column prop="sellingPrice" label="售价"></el-table-column>
    </el-table>
  </div>
</template>

<script>
import {onMounted, reactive,toRefs,ref,computed} from 'vue';
import {useRoute} from "vue-router";
import axios from "../utils/axios";

export default {
  name: 'OrderDetail',
  setup() {
    const route = useRoute();
    const itemList=ref([])
    const totalPrice = ref(0);
    const orderStatusString = ref("");
        // orderInfo:{},//整个相应数据（不用）
    const orderNo = ref("");
    const orderAddressInfo = ref({});
    // const loading = ref(false);
    const payTypeString = ref("");
    const payTime = ref("");
    const createTime = ref("");
    const userAddress = ref("");


    const  state=reactive({
      // loading:false,
      // orderId:0
      orderNo1: "",
      loading:false,
      result : {}
    })

    onMounted(  () => {
      // state.orderId=route.query.orderId
      const orderId =  route.query.orderId
      console.log("onmounted 里的 orderid" + orderId)
      // await getOrderDetailByOrderId(orderId1)
      getDetail(orderId)
    })


    const getDetail = (id) => {
      state.loading = true;
      // 发起获取订单详情的请求，根据需要调整请求的方式和路径
       axios.get(`/orders/${id}`)
           .then( res => {
         // const data =  res.data
         state.result=res
         console.log("i am result   "+state.result)
         orderNo.value = res.orderNo;
         console.log("i am data   "+orderNo)
         totalPrice.value = res.totalPrice;
         orderStatusString.value = res.orderStatusString;
         payTypeString.value = res.payTypeString;
         payTime.value = res.payTime;
         createTime.value = res.createTime;
         orderAddressInfo.value = res.orderAddressVO
         userAddress.value = orderAddressInfo.provinceName
                 + orderAddressInfo.cityName
                 + orderAddressInfo.regionName
                 + orderAddressInfo.detailAddress;
         itemList.value = res.newBeeMallOrderItemVOS;
         state.loading = false;
       });
    };


    return {
      ...toRefs(state),
      getDetail,
      itemList,
      totalPrice,
      orderStatusString,
      orderNo,
      orderAddressInfo,
      payTypeString,
      payTime,
      createTime,
      userAddress
    };
  },
};

</script>

<style scoped>

.category-container {
  min-height: 100%;
}

.el-descriptions {
  margin-top: 20px;
}
.cell-item {
  display: flex;
  align-items: center;
}
.margin-top {
  margin-top: 20px;
}
</style>