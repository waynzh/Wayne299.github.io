---
title: Composition Api - 组合式API
date: 2021-8-31 14:20:22
categories: FE
tags: [vue]
---

# Composition Api
解决碎片化导致理解和维护复杂组件变得困难。将同一个逻辑关注点相关代码收集在一起


## `setup`选项
- 在组件创建之前执行，一旦 props 被解析，就将作为组合式 API 的入口。
  > 避免使用 `this`，因为其发生在 `data`、`computed`或 `methods`被解析之前,还没有找到组件实例
- `setup` 是接收 `props` 和 `context` 的函数，`return`的所有内容都暴露给组件的其余部分

<!--more-->

```js
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch, toRefs, computed } from 'vue'

export default {
    component: {RepositoriesFilters, RepositoriesSortBy},
    props: {
        user: {
            type: string,
            required: true,
        }
    },
    setup(props) {
        // 使用 `toRefs` 创建对prop的 `user` property 的响应式引用
        const { user } = toRefs(props)

        // let repositories = [] // 直接定义是非响应性的
        // ref 包裹后变成响应式
        let repositories = ref([]) 
        // 利用 `.value`访问值
        const getUserRepositories = async () => {
        // 更新为 `props.user` 到 `user.value`
        //   repositories.value = await fetchUserRepositories(props.user)
          repositories.value = await fetchUserRepositories(user.value)
        }

        // 在 `mounted` 时调用 `getUserRepositories`
        onMounted(getUserRepositories) 

        // user改变时，调用 `getUserRepositories`
        watch(user, getUserRepositories)

        // 搜索功能移到 setup 中
        const searchQuery = ref("")
        const repositoriesMatchingSearchQuery = computed(() => {
          return repositories.value.filter( repository => repository.name includes(searchQuery.value) )
        })

        return {
          repositories,
          getUserRepositories, // 返回的函数与方法的行为相同
          searchQuery,
          repositoriesMatchingSearchQuery
        }
    },
    
}
```

## 带 `ref` 的响应式变量
- 加ref变为响应式。之后用`.value`访问值

## 在 `setup` 内注册生命周期钩子
- 前缀为 `on`, 即 `mounted` 看起来会像 `onMounted`
- 这些函数接受一个回调，当钩子被组件调用时，该回调将被执行

## `watch(ref[getter], cb, options)` 函数
```js
const counter = ref(0)
watch(counter, (newValue, oldValue) => {
  console.log('The new counter value is: ' + counter.value)
})

// 等效于
export default {
  data() {
    return {
      counter: 0
    }
  },
  watch: {
    counter(newValue, oldValue) {
      console.log('The new counter value is: ' + this.counter)
    }
  }
}
```

## 独立的 `computed` 属性
- 函数输出一个只读的响应式引用，用 `.value`访问值
```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

# 提取到独立的组合式函数中
```js
// src/composables/useUserRepositories.js
import { fetchUserRepositories } from '@/api/repositories'
import { ref, onMounted, watch } from 'vue'
// 该方法从外部 API 获取该用户的仓库，并在用户有任何更改时进行刷新
export default function useUserRepositories(user) {
    let repositories = ref([]) 

    const getUserRepositories = async () => {
      repositories.value = await fetchUserRepositories(user.value)
    }

    onMounted(getUserRepositories) 
    watch(user, getUserRepositories)

    return {
      repositories,
      getUserRepositories
    }
}
```

```js
// src/composables/useRepositoryNameSearch.js
import { ref, computed } from 'vue'
// 搜索功能
export default function useRepositoryNameSearch(repositories) {
  const searchQuery = ref('')
  const repositoriesMatchingSearchQuery = computed(() => {
    return repositories.value.filter(repository => {
      return repository.name.includes(searchQuery.value)
    })
  })

  return {
    searchQuery,
    repositoriesMatchingSearchQuery
  }
```

```js
// src/components/UserRepositories.vue
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import { toRefs } from 'vue'
// 使用两个单独的功能
export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: {
      type: String,
      required: true
    }
  },
  setup (props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)
    const { searchQuery, repositoriesMatchingSearchQuery } = useRepositoryNameSearch(repositories)

    return {
      // 不管未经过滤的仓库 故用`repositories`展示过滤后的结果
      repositories: repositoriesMatchingSearchQuery,
      getUserRepositories,
      searchQuery,
    }
  }
}
```

# TS
```ts
import { defineComponent } from '@vue/composition-api'

export default defineComponent({
  // type inference enabled
})
```