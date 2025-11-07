<template>
  <el-table :data="tableData" border stripe>
    <el-table-column prop="date" label="日期" width="180"></el-table-column>
    <el-table-column prop="name" label="姓名" width="180"></el-table-column>
    <el-table-column prop="address" label="地址">
      <template #default="{ row, $index }">
        <el-button type="danger" text @click="deleHand($index)">删除</el-button>
      </template>
    </el-table-column>
  </el-table>
</template>

<script setup lang="ts">
import { ElMessageBox } from 'element-plus'
import { computed, ref, watch } from 'vue'

const props = withDefaults(
  defineProps<{
    data: any[]
    _isComplete?: boolean
  }>(),
  {
    data: () => []
  }
)

const tableData = ref<Array<any>>([])
watch(
  () => props._isComplete,
  (val) => {
    if (val) {
      tableData.value = props.data ?? []
    }
  },
  {
    immediate: true
  }
)

/**
 * 删除
 * @param index 
 */
const deleHand = (index: number) => {
  ElMessageBox.confirm('确定删除吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    tableData.value.splice(index, 1)
  })
}
</script>

<style lang="scss" scoped></style>
