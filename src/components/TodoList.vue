<template>
    <div class="todoapp-container">
      <header class="header">
        <div class="search-bar">
          <input
            class="new-todo"
            autocomplete="off"
            placeholder="Type your todo"
            v-model="newTodo"
            @keyup.enter="addTodo"
          >
        </div>
      </header>
      <section class="main" v-show="todos.length" v-cloak>
        <div class="completed-wrapper">
          <input
            id="toggle-all"
            class="toggle-all"
            type="checkbox"
            v-model="allDone"
          >
          <label for="toggle-all">Complete all tasks</label>
          <button class="clear-completed" @click="removeCompleted">
            Clear completed
          </button>
        </div>
        <ul class="todo-list">
          <li
            v-for="todo in filteredTodos"
            :key="todo.id"
            :class="{ completed: todo.completed, editing: todo === editedTodo }"
          >
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label v-if="todo !== editedTodo" @dblclick="editTodo(todo)">{{ todo.title }}</label>
              <input
                class="edit"
                type="text"
                v-if="todo === editedTodo"
                v-model="todo.title"
                v-todo-focus="todo === editedTodo"
                @blur="doneEdit(todo)"
                @keyup.enter="doneEdit(todo)"
                @keyup.esc="cancelEdit(todo)"
              >
            </div>
          </li>
        </ul>
      </section>
      <footer class="footer" v-show="todos.length" v-cloak>
      <ul class="filters">
        
          <a @click="setVisibility('all')" :class="{ selected: visibility === 'all' }">
            All
          </a>
           <a @click="setVisibility('active')" :class="{ selected: visibility === 'active' }">
            Uncompleted
          </a>
        
           <a @click="setVisibility('completed')" :class="{ selected: visibility === 'completed' }">
            Completed
          </a>
       </ul>
        <div class="todo-count">
          <strong>{{ remaining }}</strong> {{ pluralize(remaining) }} left
        </div>
      </footer>
      <div class="no-todos" v-if="todos.length === 0">
        Type your first task, sir
      </div>
      <button
        class="new-todo-button"
        @click="addTodo"
        :class="{ inactive: newTodo.length === 0 }"
      >
        Add task
      </button>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref, reactive, computed, watchEffect } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  
  const presetValues = [
    {
      title: 'Wake up at 5am',
      completed: true,
    },
    {
      title: 'Learn how to use Vue.js',
      completed: false,
    },
    {
      title: 'Drink coffee',
      completed: false,
    },
  ];
  
  const STORAGE_KEY = 'todo-app';
  
  const todoStorage = {
    fetch() {
      const todos = JSON.parse(localStorage.getItem(STORAGE_KEY)) || presetValues;
      todos.forEach((todo, index) => {
        todo.id = index;
      });
      todoStorage.uid = todos.length;
      return todos;
    },
    save(todos) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
    },
  };
  
  const filters = {
    all(todos) {
      return todos;
    },
    active(todos) {
      return todos.filter((todo) => !todo.completed);
    },
    completed(todos) {
      return todos.filter((todo) => todo.completed);
    },
  };
  
  const todos = reactive(todoStorage.fetch());
  const newTodo = ref('');
  const editedTodo = ref(null);
  const visibility = ref('all');
  
  watchEffect(() => {
    todoStorage.save(todos);
  });
  
  const filteredTodos = computed(() => {
    return filters[visibility.value](todos);
  });
  
  const remaining = computed(() => {
    return filters.active(todos).length;
  });
  
  const allDone = computed({
    get() {
      return remaining.value === 0;
    },
    set(value) {
      todos.forEach((todo) => {
        todo.completed = value;
      });
    },
  });
  
  const addTodo = () => {
    const value = newTodo.value && newTodo.value.trim();
    if (!value) {
      return;
    }
    const existingTodo = todos.find((todo) => todo.title === value);
    if (existingTodo) {
      return;
    }
    todos.push({
      id: todoStorage.uid++,
      title: value,
      completed: false,
    });
    newTodo.value = '';
  };
  
  const removeTodo = (todo) => {
    todos.splice(todos.indexOf(todo), 1);
  };
  
  const editTodo = (todo) => {
    editedTodo.value = { ...todo, beforeEditCache: todo.title };
  };
  
  const doneEdit = (todo) => {
    if (!editedTodo.value) {
      return;
    }
    editedTodo.value = null;
    todo.title = todo.title.trim();
    if (!todo.title) {
      removeTodo(todo);
    }
  };
  
  const cancelEdit = (todo) => {
    editedTodo.value = null;
    todo.title = todo.beforeEditCache;
  };
  
  const removeCompleted = () => {
    todos.splice(0, todos.length, ...filters.active(todos));
  };
  
  const setVisibility = (value) => {
    visibility.value = value;
  };
  
  const todoFocus = (el, binding) => {
    if (binding.value) {
      el.focus();
    }
  };
  
  const pluralize = (count) => {
    return count === 1 ? 'task' : 'tasks';
  };
  </script>
   