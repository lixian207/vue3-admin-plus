<template>
  <div class="p-10px">
    <el-form v-show="showSearch" :model="queryParams" :inline="true" label-width="68px">
      <el-form-item label="用户名称" prop="menuName">
        <el-input
          v-model.trim="queryParams.menuName"
          placeholder="请输入菜单名称"
          clearable
          class="wi-240px"
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable class="wi-240px">
          <el-option v-for="dict in sys_normal_disable" :key="dict.value" :label="dict.label" :value="dict.value" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>
    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button type="primary" plain icon="Plus" @click="handleAdd">新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button type="info" plain icon="Sort" @click="toggleExpandAll">展开/折叠</el-button>
      </el-col>
      <RightToolBar v-model:showSearch="showSearch" @queryTable="getList" />
      <ColumnFilter v-if="menuList.length" :is-operation="true" :cols="tableHeadColumns" @colChange="colChange" />
    </el-row>
    <el-table
      v-if="refreshTable"
      ref="refElTable"
      v-loading="loading"
      :tree-props="{ children: 'children', hasChildren: 'hasChildren' }"
      row-key="menuId"
      :default-expand-all="isExpandAll"
      border
      :data="menuList"
      @selection-change="handleSelectionChange"
    >
      <!--column头字段-->
      <template v-for="item in tableHeadColumns">
        <el-table-column
          v-if="!item.isTemplate && item.showColumn"
          :key="item.prop"
          show-overflow-tooltip
          v-bind="item"
          :align="item.align || 'center'"
          :prop="item.prop"
          :label="item.label"
        />
        <!--状态-->
        <el-table-column
          v-if="item.prop === 'status' && item.showColumn"
          :key="item.prop"
          v-bind="item"
          align="center"
          :prop="item.prop"
          :label="item.label"
        >
          <template #default="{ row }">
            <DictTag :options="sys_normal_disable" :value="row.status" />
          </template>
        </el-table-column>
        <!--状态-->
        <el-table-column
          v-if="item.prop === 'icon' && item.showColumn"
          :key="item.prop"
          v-bind="item"
          align="center"
          :prop="item.prop"
          :label="item.label"
        >
          <template #default="{ row }">
            <svg-icon :icon-class="row.icon" />
          </template>
        </el-table-column>
      </template>

      <el-table-column label="操作" align="center" width="150" class-name="small-padding fixed-width">
        <template #default="{ row }">
          <el-tooltip content="修改" placement="top">
            <el-button link type="primary" icon="Edit" size="large" @click="handleUpdate(row)" />
          </el-tooltip>
          <el-tooltip content="新增" placement="top">
            <el-button link type="primary" icon="Plus" size="large" @click="handleAddByParent(row)" />
          </el-tooltip>
          <el-tooltip content="删除" placement="top">
            <el-button link type="primary" icon="Delete" size="large" @click="handleDelete(row)" />
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <AddEditModal ref="refAddEditModal" @getList="getList" />
  </div>
</template>
<script setup>
import ColumnFilter from '@/components/ColumnFilter.vue'
import RightToolBar from '@/components/RightToolBar.vue'
import AddEditModal from './AddEditModal.vue'
import DictTag from '@/components/DicTag.vue'
import { listReq } from '@/api/menu'
import { useDict } from '@/hooks/use-dict'
import { computed, onMounted, reactive, ref, watch } from 'vue'
/*查询模块*/
const queryParams = reactive({
  menuName: '', //菜单名称
  status: '' //状态
})
const isExpandAll = ref(false)
const refreshTable = ref(true)
function toggleExpandAll() {
  refreshTable.value = false
  isExpandAll.value = !isExpandAll.value
  nextTick(() => {
    refreshTable.value = true
  })
}
//备份数据
const bakQueryParams = JSON.stringify(queryParams)
const dateRange = ref([])
//查询
const handleQuery = () => {
  getList(queryParams)
}
//重置
const resetQuery = () => {
  resetData(queryParams, bakQueryParams)
  handleQuery()
}

const handleAddByParent = ({ menuId }) => {
  refAddEditModal.value.showModal({ menuId:"", parentId: menuId })
}

const handleUpdate = (row) => {
  const menuId = row.menuId || ids[0]
  refAddEditModal.value.showModal({ menuId })
}
const getList = () => {
  loading.value = true
  if (dateRange.value.at(-1)) {
    queryParams.beginTime = dateRange.value.at(-1)
    queryParams.endTime = dateRange.value.at(-2)
  }
  listReq(removeEmptyKey(queryParams)).then(({ data }) => {
    loading.value = false
    menuList.value = handleTree(data, 'menuId')
  })
}
onMounted(() => {
  handleQuery()
})
//字典数据
// eslint-disable-next-line camelcase
const { sys_normal_disable } = useDict('sys_normal_disable')

//导入当前页面封装方法
import {
  colChange,
  currentHook,
  handleAdd,
  handleImport,
  handleSelectionChange,
  handleTree,
  removeEmptyKey
} from './index-hook'
import { resetData } from '@/hooks/use-common'
const {
  refAddEditModal,
  refElTable,
  single,
  ids,
  totalNum,
  loading,
  menuList,
  showSearch,
  tableHeadColumns,
  handleDelete
} = currentHook(queryParams, getList)
</script>
