<script>
import { TransitionGroup } from 'vue';
import StatusFilter from './components/StatusFilter.vue';
import TodoItem from './components/TodoItem.vue';
import { createTodo, deleteTodo, getTodos, updateTodo } from './api/todos';
import Message from './components/Message.vue';

export default {
  components: {
    StatusFilter,
    TodoItem,
    Message,
  }, 
  data() {
     return {
      todos: [],
      title: '',
       status: 'all',
      errorMessage: '',
    }
  },
  computed: {
    activeTodos() {
      return this.todos.filter(todo => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter(todo => todo.completed);
    },
    visibleTodos() {
      switch (this.status) {
        case 'active':
          return this.activeTodos;
        case 'completed':
          return this.completedTodos;
        default:
          return this.todos;
      }
    }
  },
  mounted() {
    getTodos().then(({ data }) => {
      this.todos = data;
    }).catch(() => this.errorMessage = 'Unable to load todos');
  }, 
  methods: {
    handleSubmit() {
      if (this.title === '') {
        this.errorMessage = 'Title should not be empty';
        return;
      }

      createTodo(this.title).then(({ data }) => {
        this.todos = [...this.todos, data];
        this.title = '';
      }).catch(() => this.errorMessage = 'Unable to create todo');
    },
    updateTodo({id, title, completed}) {
      updateTodo({ id, title, completed }).then(({ data }) => {
        this.todos = this.todos.map(
          todo => todo.id !== id ? todo : data
        );
      }).catch(() => this.errorMessage = 'Unable to edit todo');
    },
    deleteTodo(todoId) {
      deleteTodo(todoId).then(() => {
        this.todos = this.todos.filter(todo => todo.id !== todoId)
      }).catch(() => this.errorMessage = 'Unable to delete todo');
    },
    clearCompleted() {
      Promise.allSettled(
        this.completedTodos.map(todo => deleteTodo(todo.id)))
        .then(this.todos = this.activeTodos)
        .catch(() => this.errorMessage = 'Unable to delete completed todos');
      
    }, 
    toggleAll() {
      const todosToToggle =
      this.completedTodos.length === this.todos.length
        ? this.todos
          : this.todos.filter(todo => !todo.completed);

      Promise.allSettled(todosToToggle.map(todo => updateTodo({ id: todo.id, completed: !todo.completed, title: todo.title }))).then(this.todos = this.todos.map(todo => {
        if (todosToToggle.some(t => t.id === todo.id)) {
          return { ...todo, completed: !todo.completed };
        }
          return todo;
        })).catch(() => this.errorMessage = 'Unable to toggle todos');
    }
  },
}
</script>
<template>
      <div class="todoapp">
      <h1 class="todoapp__title">Todos</h1>

      <div class="todoapp__content">
        <header class="todoapp__header">
          <button
            type="button"
            class="todoapp__toggle-all"
            :class="{active: activeTodos.length === 0}"
            @click="toggleAll"
          ></button>

          <form @submit.prevent="handleSubmit">
            <input
              type="text"
              class="todoapp__new-todo"
              placeholder="What needs to be done?"
              v-model="title"
            />
          </form>
        </header>

        <TransitionGroup name="list" tag="section" class="todoapp__main">
          <TodoItem 
            v-for="todo, index of visibleTodos" 
            :key="todo.id"
            :todo="todo"
            @update="updateTodo"
            @delete="deleteTodo(todo.id)"
            />
        </TransitionGroup>

        <footer class="todoapp__footer">
          <span class="todo-count">
            {{ activeTodos.length }} items left
          </span>

          <StatusFilter v-model="status"/>
          <button
            type="button"
            class="todoapp__clear-completed"
            :disabled="completedTodos.length === 0"
            @click="clearCompleted"
          >
            Clear completed
          </button>
        </footer>
      </div>

      <Message class="is-warning" :active="errorMessage !== ''" @hide="errorMessage =''">
        <p>{{ errorMessage }}</p>
        <template #header>
          <p>Server Error</p>
        </template>
      </Message>    
    </div>
</template>
<style>
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>

