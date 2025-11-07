<template>
  <div class="chat-menu-wrap">
    <div class="chat-menu-box">
      <div class="chat-input--wrap">
        <el-input
          type="textarea"
          @keydown="keyEnterSubmitHand"
          placeholder=""
          :autosize="{
            minRows: 1,
            maxRows: 3
          }"
          resize="none"
          class="chat-input"
          maxlength="500"
          v-model="modelValueCom"
        ></el-input>
      </div>

      <!-- 菜单功能 -->
      <div class="menu-wrap">
        <btnSend
          :disabled="!hasContent || sendLoading"
          :loading="sendLoading"
          class="btn-submit"
          :icon="Top"
          @click="submitHand"
          v-if="!receiveLoading"
        />
        <btnSend
          v-if="receiveLoading"
          :loading="sendLoading"
          class="btn-submit"
          :icon="SwitchButton"
          @click="stopReciveHand"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import { Top, SwitchButton } from '@element-plus/icons-vue'
import btnSend from '../components/btn-send.vue'
const props = withDefaults(
  defineProps<{
    // 文本内容
    modelValue: string
    // 模式 deepseek-chat, deepseek-reasoner
    model: 'deepseek-chat' | 'deepseek-reasoner'

    // 正在发送接口
    sendLoading?: boolean
    // 正在接受数据
    receiveLoading?: boolean
  }>(),
  {
    modelValue: ''
  }
)

let modelValueCom = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emits('update:modelValue', value)
  }
})

/**文本框有内容 */
let hasContent = computed(() => {
  return modelValueCom.value.trim().length > 0
})

/**
 * 键盘enter发送
 * @param e
 */
const keyEnterSubmitHand = (e: Event) => {
  const event = e as KeyboardEvent
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    submitHand()
  }
}

/**
 * 点击发送按钮
 */
const submitHand = () => {
  if (props.receiveLoading) return
  if (props.sendLoading) return
  if (!hasContent.value) return
  emits('submit', modelValueCom.value)
}

/**
 * 主动结束请求
 */
const stopReciveHand = () => {
  emits('stopRecive')
}
const emits = defineEmits(['update:modelValue', 'update:model', 'submit', 'stopRecive'])
</script>

<style lang="scss" scoped>
.chat-menu-wrap {
  background-color: var(--chat-input-bg);
  border-radius: 28px 28px 0 0;

  width: 100%;
}
.chat-menu-box {
  width: 100%;
  // min-height: 100px;
  border-radius: 28px;
  box-sizing: border-box;
  padding: 12px;
  // box-shadow: inset 0px 0px 0px 0.5px var(--chat-input-border);
  border: solid 1px var(--chat-input-border);
  // box-shadow:
  //   rgba(255, 255, 255, 0) 0px 0px 0px 0px,
  //   rgba(255, 255, 255, 0) 0px 0px 0px 0px,
  //   rgba(255, 255, 255, 0) 0px 0px 0px 0px,
  //   rgba(255, 255, 255, 0) 0px 0px 0px 0px,
  //   rgba(255, 255, 255, 0.1) 0px 10px 15px -3px,
  //   rgba(255, 255, 255, 0.1) 0px 4px 6px -4px;

  // box-shadow:

  //   inset rgba(255, 255, 255, 0.1) 0px -3px 5px -3px,
  //   inset rgba(255, 255, 255, 0.1) 0px -3px 6px -4px;
  display: flex;
  // flex-direction: column;
  position: relative;
  z-index: 99;
}
.chat-input--wrap {
  box-sizing: border-box;
  flex: 1;
  position: relative;
  color: var(--chat-input-text);
  caret-color: var(--chat-input-text);
  font-size: var(--chat-input-text-size);
}
.chat-input {
  box-sizing: border-box;
  // padding: 8px 4px 8px 8px;
  overflow: hidden;
  overflow-y: auto;
  scrollbar-color: rgb(215, 215, 215) transparent;
  ::v-deep(.el-textarea__inner) {
    border: none;
    box-shadow: none;
    scrollbar-width: thin;
    caret-color: inherit;
    line-height: 22px;
    font-size: 16px;
    &:hover {
      scrollbar-color: rgb(206, 206, 206) transparent;
    }
  }
}

.menu-wrap {
  position: relative;
  margin-top: 4px;
  z-index: 1;
  display: flex;
  align-items: center;
}
.btn-submit {
  margin-left: auto;
}
</style>
