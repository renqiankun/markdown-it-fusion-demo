<template>
  <div class="card-wrap">
    <el-input v-model="test"></el-input>
    <el-button style="margin-top: 20px;" type="primary" @click="clickHand">提交 {{ type }}</el-button>
  </div>
</template>

<script setup lang="ts">
import { ElMessage } from 'element-plus'
import { onBeforeUnmount, onMounted, ref } from 'vue'
const props = withDefaults(
  defineProps<{
    type?: any
    _data?: any
    _isComplete?: boolean
  }>(),
  {}
)

let test = ref('测试数据')

onMounted(() => {
  console.log('mounted')
})
onBeforeUnmount(() => {
  console.log('unmounted')
})

const clickHand = () => {
  ElMessage.success(test.value)
}
</script>

<style lang="scss" scoped>
.card-wrap{
  display: block;
  max-width: 400px;
  box-sizing: border-box;
  padding: 20px;
  border-radius: 8px;
  box-shadow:  0 0 8px 0 rgba(0,0,0,0.2);
  margin-top: 20px;
  margin-bottom: 20px;
}
</style>
