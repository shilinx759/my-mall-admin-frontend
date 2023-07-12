<template>
  <el-card class="category-container">
    <el-form

        label-position="left"
        label-width="100px"
        style="max-width: 460px;position: center;padding-left: 200px"
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
        <el-form-item label="地址信息">
          <el-input v-model="userAddress" disabled />
        </el-form-item>


          <el-form-item label="修改地址信息">
            <el-input  class="address-item" v-model="userName" placeholder="请输入姓名"></el-input>
            <el-divider></el-divider>
            <el-input  class="address-item" v-model="userPhone" placeholder="请输入手机号"></el-input>
            <el-cascader
                class="address-item"
                @change="handleChange"
                v-model="selectedRegion"
                :options="regions"
                :props="cascaderProps"
                :aria-placeholder="userAddress"
            ></el-cascader>
            <el-divider></el-divider>
            <el-input  class="address-item" v-model="detailAddress" placeholder="请输入详细地址"></el-input>
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
    const orderAddressVO = ref({});

    //地区选择
    const selectedRegion = ref([]);
    function handleChange(value) {
      console.log('Selected region:', value);
    }
    const cascaderProps = {
      value: 'code',
      label: 'name',
      children: 'children'
    };
    const userName = ref('');
    const userPhone = ref('');
    const detailAddress = ref('');
    const regions = [
      {
        code: '110000',
        name: '湖南省',
        children: [
          { code: '110100',
            name: '长沙市',
            children: [
              { code: '130102', name: 'XX1区' },
              { code: '130104', name: 'xx2区'}
            ]
          },
          { code: '110200',
            name: '永州市' ,
            children: [
              { code: '130102', name: '零陵区' },
              { code: '130104', name: 'xxx区'}
            ]
          }
        ]
      },
      {
        code: '130000',
        name: '河北省',
        children: [
          {
            code: '130100',
            name: '石家庄市',
            children: [
              { code: '130102', name: '长安区' },
              { code: '130104', name: '桥西区'}
            ]
          },
          {
            code: '130200',
            name: '唐山市',
            children: [
              { code: '130202', name: '路南区'},
              { code: '130204', name: '古冶区'}
            ]
          }
        ]
      },
      {
        code: '140000',
        name: '广西壮族自治区',
        children: [
          {
            code: '140100',
            name: '柳州市',
            children: [
              { code: '140102', name: '柳南区' },
              { code: '140104', name: '柳北区'}
            ]
          },
          {
            code: '140200',
            name: '南宁',
            children: [
              { code: '140202', name: '某某区'},
              { code: '140204', name: '某某2区'}
            ]
          },
          {
            code: '140300',
            name: '桂林',
            children: [
              { code: '140302', name: '雁山区'},
              { code: '140304', name: '七星区'}
            ]
          }
        ]
      }
      // 其他省市区数据...
    ];

    const  state=reactive({
      orderNo1: "",
      loading:false,
      result : {},
      error:"",
      success: {}
    })

    onMounted(  () => {
      getDetail(route.query.orderId)
    })

    const editDetail=()=>{//点击表单提交按钮，传递对应参数对象
      state.loading = true;
      orderAddressVO.value = {
        orderId: orderId.value,
        userName: userName.value,
        userPhone: userPhone.value,
        provinceName: regions.find(region => region.code === selectedRegion.value[0])?.name,
        cityName: regions.find(region => region.code === selectedRegion.value[0])?.children.find(city => city.code === selectedRegion.value[1])?.name,
        regionName: regions.find(region => region.code === selectedRegion.value[0])?.children.find(city => city.code === selectedRegion.value[1])?.children.find(region => region.code === selectedRegion.value[2])?.name,
        detailAddress: detailAddress.value
      };
      console.log("地址对象： "+orderAddressVO.toString())

      axios.put(`/orders/updateDetail`,{
          orderId:orderId.value,
          orderNo: orderNo.value,
          totalPrice:totalPrice.value,
          payStatus:orderInfo.value.payStatus,
          payTypeString:orderInfo.value.payTypeString,
          orderStatus: orderInfo.value.orderStatus,
          orderStatusString: orderStatusString.value,
          createTime: orderInfo.value.createTime,
          newBeeMallOrderItemVOS:itemList.value,
          orderAddressVO:orderAddressVO.value
      }).then(res=>{
        console.log("修改完成")
        ElMessage.success("修改完成")
        state.success = res;
        // getDetail(orderId.value)
        // state.loading = false;
      }).catch(error=>{
        state.error=error.message
        console.log("返回的状态码失败状态码："+error.resultCode)
      })
      if (state.success !== null) {
        getDetail(orderId.value)
      }
      state.loading = false;
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
             userAddress.value = res.orderAddressVO.provinceName
                 + res.orderAddressVO.cityName
                 + res.orderAddressVO.regionName
                 + res.orderAddressVO.detailAddress;
             // orderAddressInfo.value = res.orderAddressVO
             userName.value=res.orderAddressVO.userName
             detailAddress.value=res.orderAddressVO.detailAddress
             userPhone.value=res.orderAddressVO.userPhone

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
      deleteItem,
      selectedRegion,
      regions,
      cascaderProps,
      userName,
      userPhone,
      detailAddress,
      handleChange,
      orderAddressVO
    };
  },
};

</script>

<style scoped>

.category-container {
  min-height: 100%;
}

el-cascader{
  margin-bottom: 10px;
}

.address-item  {
  margin-bottom: 10px;
}
</style>