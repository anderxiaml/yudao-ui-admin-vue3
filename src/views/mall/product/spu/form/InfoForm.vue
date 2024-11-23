<!-- 商品发布 - 基础设置 -->
<template>
  <el-form ref="formRef" :disabled="isDetail" :model="formData" :rules="rules" label-width="120px">
    <el-form-item label="菜品名称" prop="name">
      <el-input v-model="formData.name" class="w-80!" placeholder="请输入菜品名称" />
    </el-form-item>
    <el-form-item label="菜品分类" prop="categoryId">
      <el-cascader
        v-model="formData.categoryId"
        :options="categoryList"
        :props="defaultProps"
        class="w-80"
        clearable
        filterable
        placeholder="请选择菜品分类"
      />
    </el-form-item>
    <el-form-item label="菜品规格" prop="categoryId">
      <el-cascader
        v-model="formData.categoryId"
        :options="categoryList"
        :props="defaultProps"
        class="w-80"
        clearable
        filterable
        placeholder="请选择菜品规格"
      />
    </el-form-item>
    <el-form-item label="菜品价格" prop="price">
      <el-input-number
        v-model="formData.price"
        :min="0"
        :precision="2"
        :step="0.1"
        class="w-100%"
        controls-position="right"
      />
    </el-form-item>
    <el-form-item label="菜品状态" prop="status">
      <el-select v-model="formData.status" class="w-80!" placeholder="请选择状态">
        <el-option
          v-for="dict in getIntDictOptions(DICT_TYPE.PRODUCT_SPU_STATUS)"
          :key="dict.value"
          :label="dict.label"
          :value="dict.value"
        />
      </el-select>
    </el-form-item>
    <el-form-item label="菜品图" prop="picUrl">
      <UploadImg v-model="formData.picUrl" :disabled="isDetail" height="80px" />
    </el-form-item>
  </el-form>
</template>
<script lang="ts" setup>
import { PropType } from 'vue'
import { copyValueToTarget } from '@/utils'
import { propTypes } from '@/utils/propTypes'
import { defaultProps, handleTree } from '@/utils/tree'
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'
import type { Spu } from '@/api/mall/product/spu'
import * as ProductCategoryApi from '@/api/mall/product/category'
import { CategoryVO } from '@/api/mall/product/category'
import * as ProductBrandApi from '@/api/mall/product/brand'
import { BrandVO } from '@/api/mall/product/brand'

defineOptions({ name: 'ProductSpuInfoForm' })
const props = defineProps({
  propFormData: {
    type: Object as PropType<Spu>,
    default: () => {}
  },
  isDetail: propTypes.bool.def(false) // 是否作为详情组件
})

const message = useMessage() // 消息弹窗

const formRef = ref() // 表单 Ref
const formData = reactive<Spu>({
  name: '', // 商品名称
  categoryId: undefined, // 商品分类
  price: 0,
  status: undefined,
  keyword: '', // 关键字
  picUrl: '', // 商品封面图
  sliderPicUrls: [], // 商品轮播图
  introduction: '', // 商品简介
  brandId: undefined // 商品品牌
})
const rules = reactive({
  name: [required],
  categoryId: [required],
  price: [required],
  status: [required],
  keyword: [required],
  introduction: [required],
  picUrl: [required],
  sliderPicUrls: [required],
  brandId: [required]
})

/** 将传进来的值赋值给 formData */
watch(
  () => props.propFormData,
  (data) => {
    if (!data) {
      return
    }
    copyValueToTarget(formData, data)
  },
  {
    immediate: true
  }
)

/** 表单校验 */
const emit = defineEmits(['update:activeName'])
const validate = async () => {
  if (!formRef) return
  try {
    await unref(formRef)?.validate()
    // 校验通过更新数据
    Object.assign(props.propFormData, formData)
  } catch (e) {
    message.error('【基础设置】不完善，请填写相关信息')
    emit('update:activeName', 'info')
    throw e // 目的截断之后的校验
  }
}
defineExpose({ validate })

/** 初始化 */
const brandList = ref<BrandVO[]>([]) // 商品品牌列表
const categoryList = ref<CategoryVO[]>([]) // 商品分类树
onMounted(async () => {
  // 获得分类树
  const data = await ProductCategoryApi.getCategoryList({})
  categoryList.value = handleTree(data, 'id')
  // 获取商品品牌列表
  brandList.value = await ProductBrandApi.getSimpleBrandList()
})
</script>
