<template>
  <el-card class="category-container">
    <el-form
        label-position="left"
        label-width="100px"
        style="max-width: 460px"
    >
      <el-form-item label="订单号">
        <el-input v-model="orderNo" />
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
        <el-input v-model="orderAddressInfo.userName"  />
      </el-form-item>
      <el-form-item label="地址">
        <el-input v-model="userAddress" disabled/>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-check" @click="editDetail">提交修改</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="itemList" v-loading="loading" :height="200" border>
      <el-table-column prop="orderItemId" label="商品订单ID"></el-table-column>
      <el-table-column prop="goodsId" label="商品ID"></el-table-column>
      <el-table-column prop="goodsCount" label="商品数量"></el-table-column>
      <el-table-column prop="goodsName" label="商品名称"></el-table-column>
      <el-table-column prop="sellingPrice" label="售价"></el-table-column>
      <el-table-column
          label="操作"
          width="120"
      >
          <template v-slot="{row}">
            <a style="cursor: pointer" @click="deleteItem(row.orderItemId)">删除</a>
          </template>
      </el-table-column>
    </el-table>
  </el-card>
</template>

<script>
import {onMounted, reactive,toRefs,ref,computed} from 'vue';
import {useRoute} from "vue-router";
import axios from "../utils/axios";
import {ElMessage} from "element-plus";

export default {
  name: 'OrderDetail',
  setup() {
    const route = useRoute();
    const itemList=ref([])
    const totalPrice = ref(0);
    const orderStatusString = ref("");
    const orderNo = ref("");
    const orderAddressInfo = ref({});
    const payTypeString = ref("");
    const payTime = ref("");
    const createTime = ref("");
    const userAddress = ref("");
    const orderId = ref(0);
    const orderInfo = ref({});

    const  state=reactive({
      orderNo1: "",
      loading:false,
      result : {}
    })

    onMounted(  () => {
      getDetail(route.query.orderId)
    })

    const editDetail=()=>{//点击表单提交按钮，传递对应参数对象
      state.loading = true;
      axios.put(`/orders/updateDetail`,{
          orderId:orderId.value,
          orderNo: orderNo.value,
          totalPrice:totalPrice.value,
          payStatus:orderInfo.value.payStatus,
          payTypeString:orderInfo.value.payTypeString,
          orderStatus: orderInfo.value.orderStatus,
          orderStatusString: orderStatusString.value,
          createTime: orderInfo.value.createTime,
          // newBeeMallOrderItemVOS:itemList.value,//不懂拿不拿得了
          orderAddressVO:orderAddressInfo.value
      }).then(res=>{
        console.log("修改完成")
        ElMessage.success("修改完成")
        getDetail(orderId.value)
        state.loading = false;
      })
    }

    const deleteItem = (orderItemId) => {
      state.loading = true;
      axios.get(`/orders/deleteItem`,{
        //get 方式的专有属性，添加到请求地址末尾
          params:{orderItemId:orderItemId}
      }).then(res=>{
        getDetail(orderId.value)
        ElMessage.success("修改完成")
        state.loading = false;
      })
    }


    const getDetail = (id) => {
      state.loading = true;
      // 发起获取订单详情的请求，根据需要调整请求的方式和路径
       axios.get(`/orders/${id}`)
           .then( res => {
         // const data =  res.data
             state.result = res;
             orderInfo.value = res;
             console.log("i am result   "+state.result)
             orderNo.value = res.orderNo;
             console.log("i am data   "+orderNo)
             orderId.value=res.orderId
             totalPrice.value = res.totalPrice;
             orderStatusString.value = res.orderStatusString;
             payTypeString.value = res.payTypeString;
             payTime.value = res.payTime;
             createTime.value = res.createTime;
             orderAddressInfo.value = res.orderAddressVO
             userAddress.value = orderAddressInfo.value.provinceName
                     + orderAddressInfo.value.cityName
                     + orderAddressInfo.value.regionName
                     + orderAddressInfo.value.detailAddress;
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
      userAddress,
      orderId,
      orderInfo,
      editDetail,
      deleteItem
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