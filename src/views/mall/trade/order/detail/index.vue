<template>
  <ContentWrap>
    <!-- 订单信息 -->
    <el-descriptions>
      <el-descriptions-item label="订单编号: ">{{ formData.no }}</el-descriptions-item>
      <el-descriptions-item label="手机号: ">{{ formData?.user?.nickname }}</el-descriptions-item>
      <el-descriptions-item label="出餐员: ">{{ formData?.user?.nickname }}</el-descriptions-item>
      <el-descriptions-item label="订单状态: ">
        <dict-tag :type="DICT_TYPE.TRADE_ORDER_STATUS" :value="formData.status!" />
      </el-descriptions-item>
      <el-descriptions-item label="下单时间: ">{{ formData?.user?.nickname }}</el-descriptions-item>
    </el-descriptions>
  </ContentWrap>

  <!-- 各种操作的弹窗 -->
  <OrderDeliveryForm ref="deliveryFormRef" @success="getDetail" />
  <OrderUpdateRemarkForm ref="updateRemarkForm" @success="getDetail" />
  <OrderUpdateAddressForm ref="updateAddressFormRef" @success="getDetail" />
  <OrderUpdatePriceForm ref="updatePriceFormRef" @success="getDetail" />
</template>
<script lang="ts" setup>
import * as TradeOrderApi from '@/api/mall/trade/order'
import { fenToYuan } from '@/utils'
import { formatDate } from '@/utils/formatTime'
import { DICT_TYPE, getDictLabel, getDictObj } from '@/utils/dict'
import OrderUpdateRemarkForm from '@/views/mall/trade/order/form/OrderUpdateRemarkForm.vue'
import OrderDeliveryForm from '@/views/mall/trade/order/form/OrderDeliveryForm.vue'
import OrderUpdateAddressForm from '@/views/mall/trade/order/form/OrderUpdateAddressForm.vue'
import OrderUpdatePriceForm from '@/views/mall/trade/order/form/OrderUpdatePriceForm.vue'
import * as DeliveryExpressApi from '@/api/mall/trade/delivery/express'
import { useTagsViewStore } from '@/store/modules/tagsView'
import { DeliveryTypeEnum, TradeOrderStatusEnum } from '@/utils/constants'
import * as DeliveryPickUpStoreApi from '@/api/mall/trade/delivery/pickUpStore'
import { propTypes } from '@/utils/propTypes'

defineOptions({ name: 'TradeOrderDetail' })

const message = useMessage() // 消息弹窗

/** 获得 userType 颜色 */
const getUserTypeColor = (type: number) => {
  const dict = getDictObj(DICT_TYPE.USER_TYPE, type)
  switch (dict?.colorType) {
    case 'success':
      return '#67C23A'
    case 'info':
      return '#909399'
    case 'warning':
      return '#E6A23C'
    case 'danger':
      return '#F56C6C'
  }
  return '#409EFF'
}

// 订单详情
const formData = ref<TradeOrderApi.OrderVO>({
  logs: []
})

/** 各种操作 */
const updateRemarkForm = ref() // 订单备注表单 Ref
const remark = () => {
  updateRemarkForm.value?.open(formData.value)
}
const deliveryFormRef = ref() // 发货表单 Ref
const delivery = () => {
  deliveryFormRef.value?.open(formData.value)
}
const updateAddressFormRef = ref() // 收货地址表单 Ref
const updateAddress = () => {
  updateAddressFormRef.value?.open(formData.value)
}
const updatePriceFormRef = ref() // 订单调价表单 Ref
const updatePrice = () => {
  updatePriceFormRef.value?.open(formData.value)
}

/** 核销 */
const handlePickUp = async () => {
  try {
    // 二次确认
    await message.confirm('确认核销订单吗？')
    // 提交
    await TradeOrderApi.pickUpOrder(formData.value.id!)
    message.success('核销成功')
    // 刷新列表
    await getDetail()
  } catch {}
}

/** 获得详情 */
const { params } = useRoute() // 查询参数
const props = defineProps({
  id: propTypes.number.def(undefined), // 订单ID
  showPickUp: propTypes.bool.def(true) // 显示核销按钮
})
const id = (params.id || props.id) as unknown as number
const getDetail = async () => {
  if (id) {
    const res = (await TradeOrderApi.getOrder(id)) as TradeOrderApi.OrderVO
    // 没有表单信息则关闭页面返回
    if (!res) {
      message.error('交易订单不存在')
      close()
    }
    formData.value = res
  }
}

/** 关闭 tag */
const { delView } = useTagsViewStore() // 视图操作
const { push, currentRoute } = useRouter() // 路由
const close = () => {
  delView(unref(currentRoute))
  push({ name: 'TradeOrder' })
}

/** 复制 */
const clipboardSuccess = () => {
  message.success('复制成功')
}

/** 初始化 **/
const deliveryExpressList = ref([]) // 物流公司
const expressTrackList = ref([]) // 物流详情
const pickUpStore = ref({}) // 自提门店
onMounted(async () => {
  await getDetail()
  // 如果配送方式为快递，则查询物流公司
  if (formData.value.deliveryType === DeliveryTypeEnum.EXPRESS.type) {
    deliveryExpressList.value = await DeliveryExpressApi.getSimpleDeliveryExpressList()
    if (form.value.logisticsId) {
      expressTrackList.value = await TradeOrderApi.getExpressTrackList(formData.value.id!)
    }
  } else if (formData.value.deliveryType === DeliveryTypeEnum.PICK_UP.type) {
    pickUpStore.value = await DeliveryPickUpStoreApi.getDeliveryPickUpStore(
      formData.value.pickUpStoreId
    )
  }
})
</script>
<style lang="scss" scoped>
:deep(.el-descriptions) {
  &:not(:nth-child(1)) {
    margin-top: 20px;
  }

  .el-descriptions__title {
    display: flex;
    align-items: center;

    &::before {
      display: inline-block;
      width: 3px;
      height: 20px;
      margin-right: 10px;
      background-color: #409eff;
      content: '';
    }
  }

  .el-descriptions-item__container {
    margin: 0 10px;

    .no-colon {
      margin: 0;

      &::after {
        content: '';
      }
    }
  }
}

// 时间线样式调整
:deep(.el-timeline) {
  margin: 10px 0 0 160px;

  .el-timeline-item__wrapper {
    position: relative;
    top: -20px;

    .el-timeline-item__timestamp {
      position: absolute !important;
      top: 10px;
      left: -150px;
    }
  }

  .el-timeline-right-content {
    display: flex;
    align-items: center;
    min-height: 30px;
    padding: 10px;
    border-radius: var(--el-card-border-radius);
    background-color: var(--app-content-bg-color);

    &::before {
      position: absolute;
      top: 10px;
      left: 13px; /* 将伪元素水平居中 */
      border-color: transparent var(--app-content-bg-color) transparent transparent; /* 尖角颜色，左侧朝向 */
      border-style: solid;
      border-width: 8px; /* 调整尖角大小 */
      content: ''; /* 必须设置 content 属性 */
    }
  }

  .dot-node-style {
    position: absolute;
    left: -5px;
    display: flex;
    width: 20px;
    height: 20px;
    font-size: 10px;
    color: #fff;
    border-radius: 50%;
    justify-content: center;
    align-items: center;
  }
}
</style>
