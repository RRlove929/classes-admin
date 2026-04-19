<template>
  <div class="dashboard-container">
    <el-row :gutter="20">
      <el-col v-if="hasPermission('stat:data')" :span="13">
        <stat-data />
      </el-col>
      <el-col :span="11">
        <el-card class="info">
          <template #header>系统简介</template>
          <div class="info-body">
            <div class="info-content">
              <div> 可信匿名评价发布</div>
              <div class="mt-2">评价真实性验证</div>
              <div class="mt-2"> 数据查询与分析</div>
            </div>
          </div>
        </el-card>
      </el-col>
    </el-row>

    <div v-if="hasPermission('stat:login')">
      <stat-login />
    </div>
    <div v-if="hasPermission('stat:vod')">
      <stat-vod />
    </div>
  </div>
</template>
<script setup>
  import StatVod from './StatVod.vue'
  import StatLogin from './StatLogin.vue'
  import StatData from './StatData.vue'
  import { hasPermission } from '@/utils/permission.js'
</script>
<style lang="scss" scoped>
  .info {
    padding: 15px;
    border-radius: 5px;
    font-size: 16px;
    .info-body {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .info-content {
      div {
        line-height: 30px;
      }
    }
  }
</style>
