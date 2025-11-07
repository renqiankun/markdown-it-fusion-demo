<template>
  <div class="chat-wrap">
    <div class="right-chat-box">
      <el-scrollbar always ref="scrollbarRef">
        <!-- 预览 -->
        <div class="chat--box">
          <chatPreview  :messages="messages"></chatPreview>
        </div>
      </el-scrollbar>

      <!-- 输入框 -->
      <div class="right-bottom-input">
        <chatInput
          v-model="chatMessage"
          v-model:model="model"
          @submit="submitHand"
        ></chatInput>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import chatInput from './chat-input/chat-input.vue'
import chatPreview from './chat-preview/chat-preview.vue'
import { computed, onMounted, onUnmounted, ref } from 'vue'

// 所有的消息列表
let modelValue = defineModel({
  type: Array<any>,
  default: () => []
})
// 输入的文本内容
let chatMessage = ref('')
// 模型选择chat | R1
let model = ref<'deepseek-chat' | 'deepseek-reasoner'>('deepseek-chat')

//包括历史列表
let messages = computed({
  get() {
    return modelValue.value
  },
  set(value) {
    modelValue.value = value
  }
})

/**提交 */
const submitHand = async () => {
  messages.value.push({
    // role: 'user',
    content: chatMessage.value
  })
  // 清除输入框
  chatMessage.value = ''
}

onMounted(() => {
})
</script>

<style lang="scss" scoped>
$chat-width: 100%;
.chat-wrap {
  width: 100%;
  height: 100%;
  max-width: 600px;
  margin: 20px auto;
  height: calc(100vh - 40px);
  display: flex;
  box-sizing: border-box;
  padding: 10px 20px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.15);
}
.right-chat-box {
  flex: 1;
  overflow: hidden;
  width: 100%;
  box-sizing: border-box;
  padding-right: 5px;
  display: flex;
  flex-direction: column;
  & > ::v-deep(.el-scrollbar) {
    flex: 1;
    overflow: hidden;
    // 非固定像素时添加
    width: calc($chat-width + 5px);
    .el-scrollbar__view {
      width: $chat-width;
      margin: 0 auto;
      max-width: 100%;
      min-height: 100%;
      display: flex;
      flex-direction: column;
      padding-right: 15px;
      // 非固定像素时添加
      box-sizing: border-box;
    }
  }
  .chat--box {
    flex: 1;
    box-sizing: border-box;
    margin-bottom: 20px;
  }
}
.right-bottom-input {
  position: relative;
  margin: 0 auto;
  min-height: 90px;
  width: $chat-width;
  z-index: 1;
}

.right-bottom-input-desc {
  width: $chat-width;
  bottom: 0;
  height: 16px;
  font-size: 12px;
  text-align: center;
  // background-color: var(--chat-input-bg);
  margin: 0 auto;
}

.to-bottom-container {
  // margin-bottom: 20px;
  display: flex;
  justify-content: flex-end;
  position: absolute;
  right: 0;
  top: -60px;
}
</style>
