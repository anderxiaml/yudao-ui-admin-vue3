<!-- 商品中心 - 商品列表  -->
<template>
  <!-- <doc-alert title="【商品】商品 SPU 与 SKU" url="https://doc.iocoder.cn/mall/product-spu-sku/" /> -->

  <!-- 搜索工作栏 -->
  <ContentWrap>
    <el-form
      ref="queryFormRef"
      :inline="true"
      :model="queryParams"
      class="-mb-15px"
      label-width="68px"
    >
      <el-form-item label="菜品名称" prop="name">
        <el-input
          v-model="queryParams.name"
          class="!w-240px"
          clearable
          placeholder="请输入菜品名称"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="菜品分类" prop="categoryId">
        <el-cascader
          v-model="queryParams.categoryId"
          :options="categoryList"
          :props="defaultProps"
          class="w-1/1"
          clearable
          filterable
          placeholder="请选择菜品分类"
        />
      </el-form-item>
      <el-form-item label="菜品规格" prop="categoryId">
        <el-cascader
          v-model="queryParams.categoryId"
          :options="categoryList"
          :props="defaultProps"
          class="w-1/1"
          clearable
          filterable
          placeholder="请选择菜品规格"
        />
      </el-form-item>
      <el-form-item label="菜品价格" prop="name">
        <el-input
          v-model="queryParams.name"
          class="!w-150px"
          clearable
          placeholder="请输入最小值"
          @keyup.enter="handleQuery"
        />
        &nbsp;&nbsp;-&nbsp;&nbsp;
        <el-input
          v-model="queryParams.name"
          class="!w-150px"
          clearable
          placeholder="请输入最大值"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="菜品状态" prop="status">
        <el-select v-model="queryParams.status" class="!w-240px" clearable placeholder="全部">
          <el-option
            v-for="dict in getIntDictOptions(DICT_TYPE.PRODUCT_SPU_STATUS)"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="handleQuery">
          <Icon class="mr-5px" icon="ep:search" />
          搜索
        </el-button>
        <el-button @click="resetQuery">
          <Icon class="mr-5px" icon="ep:refresh" />
          重置
        </el-button>
        <el-button
          v-hasPermi="['product:spu:create']"
          plain
          type="primary"
          @click="openForm(undefined)"
        >
          <Icon class="mr-5px" icon="ep:plus" />
          新增菜品
        </el-button>
        <el-button
          v-hasPermi="['product:spu:create']"
          plain
          type="primary"
          @click="openForm(undefined)"
        >
          <Icon class="mr-5px" icon="ep:plus" />
          新增菜品分类
        </el-button>
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表 -->
  <ContentWrap>
    <el-table v-loading="loading" :data="list">
      <el-table-column label="序号" min-width="140" prop="id" />
      <el-table-column label="菜品编号" min-width="140" prop="id" />
      <el-table-column label="菜品图片" min-width="300">
        <template #default="{ row }">
          <div class="flex">
            <el-image
              fit="cover"
              :src="row.picUrl"
              class="flex-none w-50px h-50px"
              @click="imagePreview(row.picUrl)"
            />
            <div class="ml-4 overflow-hidden">
              <el-tooltip effect="dark" :content="row.name" placement="top">
                <div>
                  {{ row.name }}
                </div>
              </el-tooltip>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column align="center" label="菜品名称" min-width="90" prop="salesCount" />
      <el-table-column align="center" label="菜品分类" min-width="90" prop="salesCount" />
      <el-table-column align="center" label="菜品规格" min-width="90" prop="stock" />
      <el-table-column align="center" label="菜品价格" min-width="160" prop="price">
        <template #default="{ row }"> ¥ {{ fenToYuan(row.price) }}</template>
      </el-table-column>
      <el-table-column align="center" label="菜品状态" min-width="100" prop="status">
        <template #default="scope">
          <dict-tag :type="DICT_TYPE.PRODUCT_SPU_STATUS" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column
        :formatter="dateFormatter"
        align="center"
        label="创建时间"
        prop="createTime"
        width="180"
      />
      <el-table-column
        :formatter="dateFormatter"
        align="center"
        label="修改时间"
        prop="createTime"
        width="180"
      />
      <el-table-column align="center" fixed="right" label="操作" min-width="200">
        <template #default="{ row }">
          <el-button link type="primary" @click="openDetail(row.id)"> 详情 </el-button>
          <el-button
            v-hasPermi="['product:spu:update']"
            link
            type="primary"
            @click="openForm(row.id)"
          >
            编辑
          </el-button>
          <el-button
            v-hasPermi="['product:spu:delete']"
            link
            type="danger"
            @click="handleDelete(row.id)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <Pagination
      v-model:limit="queryParams.pageSize"
      v-model:page="queryParams.pageNo"
      :total="total"
      @pagination="getList"
    />
  </ContentWrap>
</template>
<script lang="ts" setup>
import { TabsPaneContext } from 'element-plus'
import { createImageViewer } from '@/components/ImageViewer'
import { dateFormatter } from '@/utils/formatTime'
import { defaultProps, handleTree, treeToString } from '@/utils/tree'
import { ProductSpuStatusEnum } from '@/utils/constants'
import { fenToYuan } from '@/utils'
import download from '@/utils/download'
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'
import * as ProductSpuApi from '@/api/mall/product/spu'
import * as ProductCategoryApi from '@/api/mall/product/category'

defineOptions({ name: 'ProductSpu' })

const message = useMessage() // 消息弹窗
const route = useRoute() // 路由
const { t } = useI18n() // 国际化
const { push } = useRouter() // 路由跳转

const loading = ref(false) // 列表的加载中
const exportLoading = ref(false) // 导出的加载中
const total = ref(0) // 列表的总页数
const list = ref<ProductSpuApi.Spu[]>([]) // 列表的数据
// tabs 数据
const tabsData = ref([
  {
    name: '出售中',
    type: 0,
    count: 0
  },
  {
    name: '仓库中',
    type: 1,
    count: 0
  },
  {
    name: '已售罄',
    type: 2,
    count: 0
  },
  {
    name: '警戒库存',
    type: 3,
    count: 0
  },
  {
    name: '回收站',
    type: 4,
    count: 0
  }
])

const queryParams = ref({
  pageNo: 1,
  pageSize: 10,
  tabType: 0,
  name: '',
  categoryId: undefined,
  status: '',
  createTime: undefined
}) // 查询参数
const queryFormRef = ref() // 搜索的表单Ref

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await ProductSpuApi.getSpuPage(queryParams.value)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}

/** 切换 Tab */
const handleTabClick = (tab: TabsPaneContext) => {
  queryParams.value.tabType = tab.paneName as number
  getList()
}

/** 获得每个 Tab 的数量 */
const getTabsCount = async () => {
  const res = await ProductSpuApi.getTabsCount()
  for (let objName in res) {
    tabsData.value[Number(objName)].count = res[objName]
  }
}

/** 添加到仓库 / 回收站的状态 */
const handleStatus02Change = async (row: any, newStatus: number) => {
  try {
    // 二次确认
    const text = newStatus === ProductSpuStatusEnum.RECYCLE.status ? '加入到回收站' : '恢复到仓库'
    await message.confirm(`确认要"${row.name}"${text}吗？`)
    // 发起修改
    await ProductSpuApi.updateStatus({ id: row.id, status: newStatus })
    message.success(text + '成功')
    // 刷新 tabs 数据
    await getTabsCount()
    // 刷新列表
    await getList()
  } catch {}
}

/** 更新上架/下架状态 */
const handleStatusChange = async (row: any) => {
  try {
    // 二次确认
    const text = row.status ? '上架' : '下架'
    await message.confirm(`确认要${text}"${row.name}"吗？`)
    // 发起修改
    await ProductSpuApi.updateStatus({ id: row.id, status: row.status })
    message.success(text + '成功')
    // 刷新 tabs 数据
    await getTabsCount()
    // 刷新列表
    await getList()
  } catch {
    // 异常时，需要重置回之前的值
    row.status =
      row.status === ProductSpuStatusEnum.DISABLE.status
        ? ProductSpuStatusEnum.ENABLE.status
        : ProductSpuStatusEnum.DISABLE.status
  }
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    // 发起删除
    await ProductSpuApi.deleteSpu(id)
    message.success(t('common.delSuccess'))
    // 刷新tabs数据
    await getTabsCount()
    // 刷新列表
    await getList()
  } catch {}
}

/** 商品图预览 */
const imagePreview = (imgUrl: string) => {
  createImageViewer({
    urlList: [imgUrl]
  })
}

/** 搜索按钮操作 */
const handleQuery = () => {
  getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value.resetFields()
  handleQuery()
}

/** 新增或修改 */
const openForm = (id?: number) => {
  // 修改
  if (typeof id === 'number') {
    push({ name: 'ProductSpuEdit', params: { id } })
    return
  }
  // 新增
  push({ name: 'ProductSpuAdd' })
}

/** 查看商品详情 */
const openDetail = (id: number) => {
  push({ name: 'ProductSpuDetail', params: { id } })
}

/** 导出按钮操作 */
const handleExport = async () => {
  try {
    // 导出的二次确认
    await message.exportConfirm()
    // 发起导出
    exportLoading.value = true
    const data = await ProductSpuApi.exportSpu(queryParams)
    download.excel(data, '商品列表.xls')
  } catch {
  } finally {
    exportLoading.value = false
  }
}

/** 获取分类的节点的完整结构 */
const categoryList = ref() // 分类树
const formatCategoryName = (categoryId: number) => {
  return treeToString(categoryList.value, categoryId)
}

/** 激活时 */
onActivated(() => {
  getList()
})

/** 初始化 **/
onMounted(async () => {
  // 解析路由的 categoryId
  if (route.query.categoryId) {
    queryParams.value.categoryId = Number(route.query.categoryId)
  }
  // 获得商品信息
  await getTabsCount()
  await getList()
  // 获得分类树
  const data = await ProductCategoryApi.getCategoryList({})
  categoryList.value = handleTree(data, 'id', 'parentId')
})
</script>
<style lang="scss" scoped>
.spu-table-expand {
  padding-left: 42px;

  :deep(.el-form-item__label) {
    width: 82px;
    font-weight: bold;
    color: #99a9bf;
  }
}
</style>
