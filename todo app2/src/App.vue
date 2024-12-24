<template>
  <div class="app">
    <header>
      <h1>My Todo List 📝</h1>
      <input type="text" v-model="searchItem" placeholder="Search tasks" />
    </header>

    <main class="todo-sections">
      <!-- Today -->
      <section class="todo-column">
        <h2>Today</h2>
        <div class="input-group">
          <input
            type="text"
            v-model="todayInputContent"
            placeholder="Add a new task"
          />
          <button class="add-btn" @click="addTodoToday">+</button>
        </div>
        <ul class="todo-list">
          <p class ="message" v-if="todayTodos.length === 0">Great! No more todos for today!</p>
          <li
            v-for="(todo, index) in filteredTodayTodos"
            :key="todo.id"
            :class="{ done: todo.done }"
          >
            <label>
              <input type="checkbox" v-model="todo.done" @change="moveToCompleted(todo)" />
              <span class="checkbox"></span>
              {{ todo.content }}
            </label>
            <div class="actions">
              <button class="edit" @click="editTodoToday(todo.id)">
                <font-awesome-icon icon="edit" style="color: #007bff;" />
              </button>
              <button class="delete" @click="deleteTodoToday(todo.id)">
                <font-awesome-icon icon="trash" style="color: #e25050;" />
              </button>
            </div>
          </li>
        </ul>
      </section>

      <!-- Weekly -->
      <section class="todo-column">
        <h2>This Week</h2>
        <div class="input-group">
          <input
            type="text"
            v-model="weeklyInputContent"
            placeholder="Add a new task"
          />
          <button class="add-btn" @click="addTodoWeekly">+</button>
        </div>
        <ul class="todo-list">
          <p class ="message" v-if="weeklyTodos.length === 0">Great! No more todos for this week!</p>
          <li
            v-for="(todo, index) in filteredWeeklyTodos"
            :key="todo.id"
            :class="{ done: todo.done }"
          >
            <label>
              <input type="checkbox" v-model="todo.done" @change="moveToCompleted(todo)" />
              <span class="checkbox"></span>
              {{ todo.content }}
            </label>
            <div class="actions">
              <button class="edit" @click="editTodoWeekly(todo.id)">
                <font-awesome-icon icon="edit" style="color: #007bff;" />
              </button>
              <button class="delete" @click="deleteTodoWeekly(todo.id)">
                <font-awesome-icon icon="trash" style="color: #e25050;" />
              </button>
            </div>
          </li>
        </ul>
      </section>

      <!-- Eventually -->
      <section class="todo-column">
        <h2>Eventually</h2>
        <div class="input-group">
          <input
            type="text"
            v-model="eventualInputContent"
            placeholder="Add a new task"
          />
          <button class="add-btn" @click="addTodoEventual">+</button>
        </div>
        <ul class="todo-list">
          <p class ="message" v-if="eventualTodos.length === 0">Great! No more todos!</p>
          <li
            v-for="(todo, index) in filteredEventualTodos"
            :key="todo.id"
            :class="{ done: todo.done }"
          >
            <label>
              <input type="checkbox" v-model="todo.done" @change="moveToCompleted(todo)" />
              <span class="checkbox"></span>
              {{ todo.content }}
            </label>
            <div class="actions">
              <button class="edit" @click="editTodoEventual(todo.id)">
                <font-awesome-icon icon="edit" style="color: #007bff;" />
              </button>
              <button class="delete" @click="deleteTodoEventual(todo.id)">
                <font-awesome-icon icon="trash" style="color: #e25050;" />
              </button>
            </div>
          </li>
        </ul>
      </section>

      <!-- Completed Tasks -->
      <section class="todo-column">
        <h2>Completed Tasks</h2>
        <ul class="todo-list">
          <p class ="message" v-if="completedTodos.length === 0">No tasks completed yet!</p>
          <li v-for="(todo, index) in completedTodos" :key="todo.id" class="complete">
            <span>{{ todo.content }}</span>
          </li>
        </ul>
      </section>
    </main>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed } from "vue";
import { library } from "@fortawesome/fontawesome-svg-core";
import { faTrash, faEdit } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
import { v4 as uuidv4 } from "uuid";

library.add(faTrash, faEdit);

export default {
  components: {
    FontAwesomeIcon,
  },
  setup() {
    const todayTodos = ref([]);
    const weeklyTodos = ref([]);
    const eventualTodos = ref([]);
    const completedTodos = ref([]);
    const todayInputContent = ref("");
    const weeklyInputContent = ref("");
    const eventualInputContent = ref("");
    const searchItem = ref("");

    const addTodoToday = () => {
      if (todayInputContent.value.trim() === "") return;
      todayTodos.value.push({
        id: uuidv4(),
        content: todayInputContent.value.trim(),
        done: false,
      });
      todayInputContent.value = "";
    };

    const addTodoWeekly = () => {
      if (weeklyInputContent.value.trim() === "") return;
      weeklyTodos.value.push({
        id: uuidv4(),
        content: weeklyInputContent.value.trim(),
        done: false,
      });
      weeklyInputContent.value = "";
    };

    const addTodoEventual = () => {
      if (eventualInputContent.value.trim() === "") return;
      eventualTodos.value.push({
        id: uuidv4(),
        content: eventualInputContent.value.trim(),
        done: false,
      });
      eventualInputContent.value = "";
    };

    const moveToCompleted = (todo) => {
      if (todo.done) {
        if (!completedTodos.value.some((t) => t.id === todo.id)) {
          completedTodos.value.push({ ...todo });
        }
      } else {
        completedTodos.value = completedTodos.value.filter((t) => t.id !== todo.id);  // Remove from completed when unchecked
      }
    };

    const deleteTodoToday = (id) => {
      todayTodos.value = todayTodos.value.filter((todo) => todo.id !== id);
      removeFromCompleted(id);
    };

    const editTodoToday = (id) => {
      const todo = todayTodos.value.find((todo) => todo.id === id);
      const updatedContent = prompt("Edit your task:", todo.content);
      if (updatedContent) todo.content = updatedContent.trim();
    };

    const deleteTodoWeekly = (id) => {
      weeklyTodos.value = weeklyTodos.value.filter((todo) => todo.id !== id);
      removeFromCompleted(id);
    };

    const editTodoWeekly = (id) => {
      const todo = weeklyTodos.value.find((todo) => todo.id === id);
      const updatedContent = prompt("Edit your task:", todo.content);
      if (updatedContent) todo.content = updatedContent.trim();
    };

    const deleteTodoEventual = (id) => {
      eventualTodos.value = eventualTodos.value.filter((todo) => todo.id !== id);
      removeFromCompleted(id);
    };

    const editTodoEventual = (id) => {
      const todo = eventualTodos.value.find((todo) => todo.id === id);
      const updatedContent = prompt("Edit your task:", todo.content);
      if (updatedContent) todo.content = updatedContent.trim();
    };

    const removeFromCompleted = (id) => {
      completedTodos.value = completedTodos.value.filter((todo) => todo.id !== id);
    };

    const filteredTodayTodos = computed(() =>
      todayTodos.value.filter((todo) =>
        todo.content.toLowerCase().includes(searchItem.value.toLowerCase())
      )
    );

    const filteredWeeklyTodos = computed(() =>
      weeklyTodos.value.filter((todo) =>
        todo.content.toLowerCase().includes(searchItem.value.toLowerCase())
      )
    );

    const filteredEventualTodos = computed(() =>
      eventualTodos.value.filter((todo) =>
        todo.content.toLowerCase().includes(searchItem.value.toLowerCase())
      )
    );

    onMounted(() => {
      try {
        todayTodos.value = JSON.parse(localStorage.getItem("todayTodos")) || [];
        weeklyTodos.value = JSON.parse(localStorage.getItem("weeklyTodos")) || [];
        eventualTodos.value = JSON.parse(localStorage.getItem("eventualTodos")) || [];
      } catch (error) {
        console.error("Failed to load data from localStorage:", error);
      }
    });

    watch(
      todayTodos,
      (newVal) => {
        localStorage.setItem("todayTodos", JSON.stringify(newVal));
      },
      { deep: true }
    );

    watch(
      weeklyTodos,
      (newVal) => {
        localStorage.setItem("weeklyTodos", JSON.stringify(newVal));
      },
      { deep: true }
    );

    watch(
      eventualTodos,
      (newVal) => {
        localStorage.setItem("eventualTodos", JSON.stringify(newVal));
      },
      { deep: true }
    );

    return {
      todayTodos,
      weeklyTodos,
      eventualTodos,
      completedTodos,
      todayInputContent,
      weeklyInputContent,
      eventualInputContent,
      addTodoToday,
      addTodoWeekly,
      addTodoEventual,
      moveToCompleted,
      deleteTodoToday,
      editTodoToday,
      deleteTodoWeekly,
      editTodoWeekly,
      deleteTodoEventual,
      editTodoEventual,
      searchItem,
      filteredTodayTodos,
      filteredWeeklyTodos,
      filteredEventualTodos,
      removeFromCompleted,
    };
  },
};
</script>
