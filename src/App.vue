<script setup>
  import { ref, computed, watchEffect } from "vue"

  const STORAGE_KEY = "vue-todomvc"

  const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed),
  }

  // state
  const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]"))
  const visibility = ref("all")
  const editedTodo = ref()

  const filteredTodos = computed(() => filters[visibility.value](todos.value))
  const remaining = computed(() => filters.active(todos.value).length)

  const onHashChange = () => {
    const route = window.location.hash.replace(/#\/?/, "")
    if (filters[route]) {
      visibility.value = route
    } else {
      window.location.hash = ""
      visibility.value = "all"
    }
  }

  // ルーティング用
  window.addEventListener("hashchange", onHashChange)
  onHashChange()

  // ストレージへの保存
  watchEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
  })

  const toggleAll = (e) => {
    todos.value.forEach((todo) => (todo.completed = e.target.checked))
  }

  const addTodo = (e) => {
    const value = e.target.value.trim()
    if (value) {
      todos.value.push({
        id: Date.now(),
        title: value,
        completed: false,
      })
      e.target.value = ""
    }
  }

  const removeTodo = (todo) => {
    todos.value.splice(todos.value.indexOf(todo), 1)
  }

  let beforeEditCache = ""
  function editTodo(todo) {
    beforeEditCache = todo.title
    editedTodo.value = todo
  }

  const cancelEdit = (todo) => {
    editedTodo.value = null
    todo.title = beforeEditCache
  }

  const doneEdit = (todo) => {
    if (editedTodo.value) {
      editedTodo.value = null
      todo.title = todo.title.trim()
      if (!todo.title) removeTodo(todo)
    }
  }

  const removeCompleted = () => {
    todos.value = filters.active(todos.value)
  }
</script>

<template>
  <section>
    <header>
      <h1 class="text-red-600">タスクリスト一覧</h1>
      <input autofocus placeholder="タスクを入力してください" @keyup.enter="addTodo" />
    </header>
    <section v-show="todos.length">
      <input id="toggle-all" type="checkbox" :checked="remaining === 0" @change="toggleAll" />
      <label for="toggle-all">Mark all as complete</label>
      <ul>
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          :class="{ completed: todo.completed, editing: todo === editedTodo }"
        >
          <div>
            <input type="checkbox" v-model="todo.completed" />
            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
            <button @click="removeTodo(todo)">削除</button>
          </div>
          <input
            v-if="todo === editedTodo"
            type="text"
            v-model="todo.title"
            @vnode-mounted="({ el }) => el.focus()"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.escape="cancelEdit(todo)"
          />
        </li>
      </ul>
    </section>
    <footer v-show="todos.length">
      <span>
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? "item" : "items" }} left</span>
      </span>
      <ul>
        <li>
          <a href="#/all" :class="{ selected: visibility === 'all' }">すべて</a>
        </li>
        <li>
          <a href="#/active" :class="{ selected: visibility === 'active' }">未完了</a>
        </li>
        <li>
          <a href="#/completed" :class="{ selected: visibility === 'completed' }">完了</a>
        </li>
      </ul>
      <button @click="removeCompleted" v-show="todos.length > remaining">全ての完了済みタスクを削除</button>
    </footer>
  </section>
</template>
