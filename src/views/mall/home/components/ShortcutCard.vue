<template>
  <el-card shadow="never">
    <template #header>
      <CardTitle title="快捷入口" />
    </template>
    <div class="flex flex-row flex-wrap gap-8 p-4">
      <div
        v-for="menu in menuList"
        :key="menu.name"
        class="h-20 w-20% flex flex-col cursor-pointer items-center justify-center gap-2"
        @click="handleMenuClick(menu.routerName)"
      >
        <div
          :class="menu.bgColor"
          class="h-48px w-48px flex items-center justify-center rounded text-white"
        >
          <Icon :icon="menu.icon" class="text-7.5!" />
        </div>
        <span>{{ menu.name }}</span>
      </div>
    </div>
  </el-card>
</template>
<script lang="ts" setup>
/** 快捷入口卡片 */
import { CardTitle } from '@/components/Card'

defineOptions({ name: 'ShortcutCard' })

const router = useRouter() // 路由

/** 菜单列表 */
const menuList = [
  {
    name: '订单管理',
    icon: 'ep:list',
    bgColor: 'bg-yellow-500',
    routerName: 'TradeOrder'
  },
  {
    name: '菜品管理',
    icon: 'fluent-mdl2:product',
    bgColor: 'bg-orange-400',
    routerName: 'ProductSpu'
  },
  {
    name: '套餐管理',
    icon: 'fluent-mdl2:product',
    bgColor: 'bg-orange-400',
    routerName: 'ProductSpu'
  },
  {
    name: '统计中心',
    icon: 'fluent-mdl2:product',
    bgColor: 'bg-orange-400',
    routerName: 'ProductSpu'
  }
]

/**
 * 跳转到菜单对应页面
 *
 * @param routerName 路由页面组件的名称
 */
const handleMenuClick = (routerName: string) => {
  router.push({ name: routerName })
}
</script>
