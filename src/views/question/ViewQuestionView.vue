<template>
  <div id="viewQuestionView">
    <a-row :gutter="[24, 24]">
      <a-col :md="12" :xs="24">
        <a-tabs default-active-key="question">
          <a-tab-pane key="question" title="题目">
            <a-card v-if="question" :title="question.title">
              <a-descriptions
                title="判题条件"
                :column="{ xs: 1, md: 2, lg: 3 }"
              >
                <a-descriptions-item label="时间限制">
                  {{ question.judgeConfig.timeLimit ?? 0 }}
                </a-descriptions-item>
                <a-descriptions-item label="内存限制">
                  {{ question.judgeConfig.memoryLimit ?? 0 }}
                </a-descriptions-item>
                <a-descriptions-item label="堆栈限制">
                  {{ question.judgeConfig.stakeLimit ?? 0 }}
                </a-descriptions-item>
              </a-descriptions>
              <a-card class="content">
                <h3>题目</h3>
                <MdViewer :value="question.content || ''" />
                <a-card>
                  {{ judgeCase }}
                </a-card>
              </a-card>

              <template #extra>
                <a-space wrap>
                  <a-tag
                    v-for="(tag, index) of question.tags"
                    :key="index"
                    color="green"
                  >{{ tag }}
                  </a-tag>
                </a-space>
              </template>
            </a-card>
          </a-tab-pane>
          <a-tab-pane key="comment" title="评论" disabled> 评论区</a-tab-pane>
          <a-tab-pane key="answer" title="答案">
            <a-card v-if="loginUser.userRole===accessEnum.ADMIN">
              <pre><code>{{ questionAnswer }}</code></pre>
            </a-card>
            <a-card v-else>
              <h2>您没有权限查看答案</h2>
            </a-card>
          </a-tab-pane>
        </a-tabs>
      </a-col>
      <a-col :md="12" :xs="24">
        <a-form :model="form" layout="inline">
          <a-form-item
            field="language"
            label="编程语言"
            style="min-width: 240px"
          >
            <a-select
              v-model="form.language"
              :style="{ width: '320px' }"
              placeholder="选择编程语言"
            >
              <a-option>java</a-option>
              <a-option>cpp</a-option>
              <a-option>go</a-option>
              <a-option>html</a-option>
            </a-select>
          </a-form-item>
        </a-form>
        <CodeEidtor
          :value="form.code as string"
          :language="form.language"
          :handle-change="changeCode"
        />
        <a-divider size="0" />
        <a-button type="primary" style="min-width: 200px" @click="doSubmit">
          提交代码
        </a-button>
      </a-col>
    </a-row>
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref, watchEffect, withDefaults, defineProps } from 'vue'
import message from '@arco-design/web-vue/es/message'
import {
  Question,
  QuestionControllerService,
  QuestionSubmitAddRequest,
  QuestionSubmitControllerService,
  QuestionVO
} from '../../../generated'
import CodeEidtor from '@/components/CodeEidtor.vue'
import { useStore } from 'vuex'
import accessEnum from '@/access/accessEnum'
import MdViewer from '@/components/MdViewer.vue'

const store = useStore()
const loginUser = store.state.user.loginUser
const judgeCase = ref()

interface Props {
  id: string;
}

const props = withDefaults(defineProps<Props>(), {
  id: () => ''
})

const question = ref<QuestionVO>()
const questionAnswer = ref<Question>()
const loadData = async () => {
  const res = await QuestionControllerService.getQuestionVoByIdUsingGet(
    props.id as any
  )
  if (res.code === 0) {
    question.value = res.data
    console.log(question)
  } else {
    message.error('加载失败，' + res.message)
  }
}
const loadDataAnswer = async () => {
  const resAnswer = await QuestionControllerService.getQuestionByIdUsingGet(
    props.id as any
  )
  if (resAnswer.code === 0) {
    questionAnswer.value = resAnswer.data?.answer
    judgeCase.value = JSON.parse(resAnswer.data?.judgeCase)
  } else {
    message.error('加载失败，' + resAnswer.message)
  }
}
const form = ref<QuestionSubmitAddRequest>({
  language: 'java',
  code: ''
})
/**
 * 提交代码
 */
const doSubmit = async () => {
  if (!question.value?.id) {
    return
  }

  const res = await QuestionSubmitControllerService.doSubmitQuestionUsingPost({
    ...form.value,
    questionId: question.value.id
  })
  if (res.code === 0) {
    message.success('提交成功')
  } else {
    message.error('提交失败,' + res.message)
  }
}

/**
 * 页面加载时，请求数据
 */
onMounted(() => {
  loadData()
  loadDataAnswer()
})

const changeCode = (value: string) => {
  form.value.code = value
}
</script>

<style>
#viewQuestionView {
  max-width: 1400px;
  margin: 0 auto;
}

#viewQuestionView .arco-space-horizontal .arco-space-item {
  margin-bottom: 0 !important;
}
</style>
