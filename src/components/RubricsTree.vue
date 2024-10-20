<template>
  <div>
    <el-checkbox v-model="showEmpty" @change="fetchRubrics"
      >Отображать пустые рубрики</el-checkbox
    >
    <el-tree
      :data="treeData"
      show-checkbox
      :props="defaultProps"
      node-key="id"
      default-expand-all
      :expand-on-click-node="false"
      :render-content="renderContent"
    ></el-tree>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, onMounted, watch } from 'vue'
import { ElTree, ElCheckbox } from 'element-plus'
import axios from 'axios'

interface Rubric {
  id: number
  title: string
  url: string
  count: number
  children?: Rubric[]
}

const showEmpty = ref(false)
const rubrics = ref<Rubric[]>([])
const checkedRubrics = ref<Set<number>>(new Set())

const treeData = computed(() => {
  const addCounts = (rubric: Rubric): Rubric => {
    const totalCount = rubric.children
      ? rubric.children.reduce(
          (sum, child) => sum + addCounts(child).count,
          rubric.count,
        )
      : rubric.count
    return { ...rubric, count: totalCount }
  }
  const data = rubrics.value.map(addCounts)
  console.log('Tree data:', data)
  return data
})

const defaultProps = {
  children: 'children',
  label: 'title',
}

// Функция для получения рубрик
const fetchRubrics = async () => {
  try {
    const response = await axios.get(
      'https://www.klerk.ru/yindex.php/v3/event/rubrics',
      {
        params: { allowEmpty: showEmpty.value ? 1 : 0 },
      },
    )
    rubrics.value = response.data
    console.log('Rubrics fetched:', rubrics.value)
  } catch (error) {
    console.error('Error fetching rubrics:', error)
  }
}

// Рендер контента для дерева
const renderContent = (h: any, { data: data }: { data: Rubric }) => {
  return h('span', [
    h('el-checkbox', {
      modelValue: checkedRubrics.value.has(data.id),
      onChange: (value: boolean) => {
        if (value) {
          checkedRubrics.value.add(data.id)
        } else {
          checkedRubrics.value.delete(data.id)
        }
        console.log(`Checkbox for rubric ${data.id} is now ${value}`)
        console.log('Checked rubrics updated:', [...checkedRubrics.value])
      },
    }),
    h(
      'a',
      { href: `https://www.klerk.ru${data.url}`, target: '_blank' },
      data.title,
    ),
    h('span', ` (${data.count} / ${data.count})`),
  ])
}

watch(checkedRubrics, newVal => {
  console.log('Checked rubrics changed:', [...newVal])
})

onMounted(() => {
  fetchRubrics()
})
</script>

<style scoped></style>
