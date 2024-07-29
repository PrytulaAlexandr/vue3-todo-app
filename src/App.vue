<script>
import { getTodos, createTodo, updateTodo, deleteTodo } from "./api/todos";
import Message from "./components/Message.vue";
import StatusFilter from "./components/StatusFilter.vue";
import TodoItem from "./components/TodoItem.vue";

export default {
  components: {
    StatusFilter,
    TodoItem,
    Message,
  },
  data() {
    return {
      todos: [],
      title: "",
      status: "all",
      errorMessage: "",
    };
  },
  computed: {
    activeTodos() {
      return this.todos.filter((todo) => !todo.completed);
    },
    completedTodos() {
      return this.todos.filter((todo) => todo.completed);
    },
    visibleTodos() {
      switch (this.status) {
        case "active":
          return this.activeTodos;

        case "completed":
          return this.completedTodos;

        default:
          return this.todos;
      }
    },
  },
  // watch: {
  //   todos: {
  //     deep: true,
  //     handler() {
  //       localStorage.setItem("todos", JSON.stringify(this.todos));
  //     },
  //   },
  // },
  mounted() {
    getTodos()
      .then(({ data }) => {
        this.todos = data;
      })
      .catch(() => {
        this.errorMessage = "Unable to load todos";
      });
  },
  methods: {
    handleSubmit() {
      createTodo(this.title).then(({ data }) => {
        this.todos = [...this.todos, data];
        this.title = "";
      });
    },
    updateTodo({ id, title, completed }) {
      updateTodo({ id, title, completed }).then(({ data }) => {
        this.todos = this.todos.map((todo) => (todo.id != id ? todo : data));
      });
    },
    deleteTodo(todoId) {
      deleteTodo(todoId).then(() => {
        this.todos = this.todos.filter((todo) => todo.id != todoId);
      });
    },
    clearCompleted() {
      const completedIds = this.completedTodos.map((todo) => todo.id);
      Promise.all(completedIds.map((id) => deleteTodo(id)))
        .then(() => {
          this.todos = this.todos.filter((todo) => !completedIds.includes(todo.id));
        })
        .catch(() => {
          this.errorMessage = "Unable to delete completed todos";
        });
    },
    toggleAll() {
      const allCompleted = this.activeTodos.length === 0;
      Promise.all(
        this.todos.map((todo) =>
          updateTodo({ ...todo, completed: !allCompleted }).then(({ data }) => data)
        )
      )
        .then((updatedTodos) => {
          this.todos = updatedTodos;
        })
        .catch(() => {
          this.errorMessage = "Unable to update todos";
        });
    },
  },
};
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button
          class="todoapp__toggle-all"
          :class="{ active: activeTodos.length === 0 }"
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
          v-for="(todo, index) of visibleTodos"
          :key="todo.id"
          :todo="todo"
          @update="updateTodo"
          @delete="deleteTodo(todo.id)"
        />
      </TransitionGroup>

      <footer class="todoapp__footer">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <StatusFilter v-model="status" />

        <button class="todoapp__clear-completed" 
          v-if="completedTodos.length > 0"
          @click="clearCompleted"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <Message 
      class="is-danger" 
      :active="errorMessage !== ''"
      @hide="errorMessage = ''"
    >
      <template #header>
        <p>Server Error</p>
      </template>

      <p>{{ errorMessage }}</p>
    </Message>
  </div>
</template>

<style>
.list-enter-active,
.list-leave-active {
  max-height: 60px;
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: scaleY(0);
  max-height: 0;
}
</style>
