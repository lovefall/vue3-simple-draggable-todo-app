<script>
import { ref, reactive, onMounted, computed, nextTick } from "vue";
import { Container, Draggable } from "vue-dndrop";
import { IconPark } from "@icon-park/vue-next/es/all";
export default {
  name: "ToDo App",
  components: {
    Container,
    Draggable,
    IconPark,
  },
  setup() {
    const newTodo = ref("");
    const todos = reactive([]);
    const todoInput = ref(null);
    const todoUpdateInput = ref(null);
    const delay = 700;
    let localSavedTodo = null;
    let timer = null;
    let clicks = 0;

    if (
      JSON.parse(localStorage.getItem("todos")) &&
      JSON.parse(localStorage.getItem("todos")).length !== 0
    ) {
      localSavedTodo = JSON.parse(localStorage.getItem("todos"));
      localSavedTodo.forEach((todo) => todos.push(todo));
    }

    const addTodo = () => {
      if (newTodo.value) {
        todos.push({
          content: newTodo.value,
          done: false,
          editMode: false,
        });
        newTodo.value = "";
        todoInput.value.focus();
      }
      saveData();
    };

    const doneTodo = (todo) => {
      todo.done = !todo.done;
      saveData();
    };

    const editTodo = async (todo) => {
      if (todo.done) return;
      todo.editMode = !todo.editMode;
      await nextTick();
      todoUpdateInput.value.focus();
    };

    const oneClick = (todo) => {
      clicks++;
      if (clicks === 1) {
        timer = setTimeout(() => {
          clicks = 0;
        }, delay);
      } else {
        clearTimeout(timer);
        editTodo(todo);
        clicks = 0;
      }
    };

    const onDrop = (dropResult) => {
      applyDrag(dropResult);
    };

    const applyDrag = (dragResult) => {
      const { removedIndex, addedIndex } = dragResult;
      if (removedIndex === null && addedIndex === null) return;
      if (!Number.isInteger(removedIndex) || !Number.isInteger(addedIndex))
        return;
      [todos[removedIndex], todos[addedIndex]] = [
        todos[addedIndex],
        todos[removedIndex],
      ];
      saveData();
    };

    const removeTodo = (index) => {
      todos.splice(index, 1);
      saveData();
    };

    const updateTodo = (e, todo) => {
      e.preventDefault();
      todo.content = e.target.value;
      todo.editMode = !todo.editMode;
      saveData();
    };

    const saveData = () => {
      const storageData = JSON.stringify(todos);
      localStorage.setItem("todos", storageData);
    };

    onMounted(() => {
      todoInput.value.focus();
    });

    const todoNum = computed(() => todos.length);

    return {
      todos,
      todoInput,
      todoUpdateInput,
      newTodo,
      addTodo,
      doneTodo,
      removeTodo,
      saveData,
      editTodo,
      updateTodo,
      todoNum,
      onDrop,
      IconPark,
      oneClick,
    };
  },
};
</script>

<template>
  <h1>ToDo List App</h1>
  <form @submit.prevent="addTodo()">
    <label for="newTodo">New ToDo </label>
    <input
      v-model="newTodo"
      ref="todoInput"
      id="newTodo"
      name="newTodo"
      autocomplete="off"
      placeholder="Add a new ToDo..."
    />
    <button>Add ToDo</button>
  </form>
  <h2>ToDo List:({{ todoNum }})</h2>
  <div id="outer" v-if="todos.length !== 0">
    <Container
      @drop="onDrop"
      drag-class="opacity-ghost"
      drop-class="opacity-ghost-drop"
    >
      <Draggable
        class="inner"
        v-for="(todo, index) in todos"
        :key="todo.content"
      >
        <input
          type="checkbox"
          :id="index"
          :checked="todo.done"
          @click="doneTodo(todo)"
        />
        <div
          @click="oneClick(todo)"
          class="draggable-item"
          v-show="!todo.editMode"
        >
          <span :class="{ done: todo.done }">
            {{ todo.content }}
          </span>
        </div>
        <div v-show="todo.editMode">
          <span v-show="todo.editMode">
            <input
              :id="index"
              :name="index"
              ref="todoUpdateInput"
              autocomplete="off"
              :value="todo.content"
              @keydown.enter="todoUpdateInput.blur()"
              @blur="updateTodo($event, todo)"
            />
          </span>
        </div>
        <div v-show="!todo.editMode" align="right">
          <icon-park
            v-show="!todo.editMode"
            @click="removeTodo(index)"
            type="delete"
            theme="outline"
            fill="#000"
            size="20"
          />
        </div>
        <div v-show="todo.editMode" align="right"></div>
      </Draggable>
    </Container>
  </div>
  <h4 v-if="todos.length === 0">Empty list.</h4>
</template>

<style lang="scss">
$border: 2px solid
  rgba(
    $color: white,
    $alpha: 0.35,
  );
$size1: 6px;
$size2: 12px;
$size3: 18px;
$size4: 24px;
$size5: 48px;
$backgroundColor: #27292d;
$textColor: white;
$primaryColor: #a0d0d9;
$borderColor: #f7e70b;
$secondTextColor: #1f2023;
body {
  margin: 0;
  padding: 0;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: $backgroundColor;
  color: $textColor;
  #app {
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
    padding: 20px;
    h1 {
      font-weight: bold;
      font-size: 28px;
      text-align: center;
    }
    form {
      display: flex;
      flex-direction: column;
      width: 100%;
      label {
        font-size: 14px;
        font-weight: bold;
      }
      input,
      button {
        height: $size5;
        box-shadow: none;
        outline: none;
        padding-left: $size2;
        padding-right: $size2;
        border-radius: $size1;
        font-size: 18px;
        margin-top: $size1;
        margin-bottom: $size2;
      }
      input {
        background-color: transparent;
        border: $border;
        color: inherit;
      }
    }
    button {
      cursor: pointer;
      background-color: $primaryColor;
      border: 1px solid $borderColor;
      color: $secondTextColor;
      font-weight: bold;
      outline: none;
      border-radius: $size1;
      margin-right: 10px;
    }
    h2 {
      font-size: 22px;
      border-bottom: $border;
      padding-bottom: $size1;
    }
    #outer {
      padding: 10px;
      .inner {
        display: flex;
        background-color: rgb(112, 112, 41);
        justify-content: space-between;
        align-items: center;
        border: $border;
        padding: $size2 $size4;
        border-radius: $size1;
        margin-bottom: $size2;
        span {
          cursor: pointer;
        }
        .done {
          text-decoration: line-through;
        }
        button {
          font-size: $size2;
          padding: $size1;
        }
      }
    }
    h4 {
      text-align: center;
      opacity: 0.5;
      margin: 0;
    }
    input[type="checkbox"] {
      display: inline-block;
      background-color: rgb(50, 124, 173);
      padding: 3px 6px;
      border: 2px solid rgb(19, 18, 18);
      color: #444;
      user-select: none;
      margin-right: 15px;
    }
    input[type="checkbox"]:checked {
      color: yellow;
      background-color: rgb(50, 124, 173);
    }
  }
}
</style>
