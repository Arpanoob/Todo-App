<script setup>
import { onMounted, ref } from 'vue'

const selectedPriority = ref(''); // Define selectedPriority as a ref variable


const loading = ref(false); // Define selectedPriority as a ref variable


const dateString = "2024-03-09T19:41:55.983Z";

const options = {
  year: "numeric",
  month: "long",
  day: "numeric",
  hour: "numeric",
  minute: "numeric",
  second: "numeric",
  timeZoneName: "short"
};


const newTodoText = ref('')
const todos = ref([])

let nextTodoId = 4


async function deleteTask(id) {
  try {
    const requestOptions = {
      method: "DELETE",
    };
    const response = await fetch(`https://vue-js-todo-backend.onrender.com/api/task/${id}`, requestOptions)

  } catch (error) {
    console.error(error);
  }
}
async function removeTodo(todo) {
  await deleteTask(todo.id)
  todos.value = todos.value.filter((t) => (t !== todo))

}
async function createTask(text) {
  try {
    const requestOptions = {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ text: text })
    };
    const response = await fetch("https://vue-js-todo-backend.onrender.com/api/task", requestOptions)
  } catch (error) {
    console.error(error);
  }
}
async function addNewTodo() {
  if (!newTodoText.value) {
    window.alert("Text Bo not to Be empty")
    return;
  }
  await createTask(newTodoText.value)
  newTodoText.value = ''
  selectedPriority.value = ''; // Reset the dropdown to "All"
  await callTheApi(selectedPriority.value); // Refresh the list based on the new filter
}
async function fetchTasks() {
  try {
    loading.value = true; // Set loading to true before fetching tasks
    const response = await fetch('https://vue-js-todo-backend.onrender.com/api/task');
    const tasks = await response.json();
    todos.value = tasks;
    loading.value = false; // Set loading to false after tasks are fetched
  } catch (e) {
    console.log(e);
    loading.value = false; // Ensure loading is set to false even if there's an error
  }
}

onMounted(async () => {

  await fetchTasks();
})
async function updateTodoStatus(todo) {
  console.log(todo)
  try {

    const requestOptions = {
      method: "PUT",
      headers: { "Content-Type": "application/json", 'Access-Control-Allow-Origin': '*' },
      body: JSON.stringify({ isCompleted: todo.isCompleted }) // Change to 'completed' instead of 'isCompleted'
    };
    loading.value = true;
    const response = await fetch(`https://vue-js-todo-backend.onrender.com/api/task/${todo.id}`, requestOptions);
    if (!response.ok) {
      throw new Error('Failed to update task status');

    } loading.value = false
    selectedPriority.value = ''; // Reset the dropdown to "All"
    await callTheApi(selectedPriority.value);
    // Handle success if necessary
  } catch (error) {
    loading.value = false
    console.error('Error updating task status:', error);
    // Handle error
  }
}

async function callTheApi(selectedPriority) {
  // Convert the string value to boolean

  loading.value = true;
  const response = await fetch(`https://vue-js-todo-backend.onrender.com/api/task/${selectedPriority}`);
  const tasks = await response.json();
  loading.value = false;
  todos.value = tasks;


}

</script>

<template>

  <div class="container">
    <div class="form-container">
      <form @submit.prevent="addNewTodo">
        <label class="heading" for="new-todo">Add a todo</label>
        <br />
        <input v-model="newTodoText" id="new-todo" class="new-todo" placeholder="E.g. Feed the cat" />
        <br />
        <button class="Add">Add</button>
        <br />
        <label class="filter" for="new-todo">Filter</label><br />
        <select class="dr" v-model="selectedPriority" @change="callTheApi(selectedPriority)">
          <option value="">All</option>
          <option value="true">Completed</option>
          <option value="false">Not Completed</option>
        </select>

      </form>
    </div>
    <div class="list-container">
      <label class="todo">
        Todo List
      </label>
      <div class="listparent">
        <div v-if="loading">Loading...</div>
        <!-- Actual list rendering -->
        <div v-else class="list">
          <ol>
            <li v-for="todo in todos" :key="todo.id">
              <div class='li'>
                <input type="checkbox" v-model="todo.isCompleted" @change="updateTodoStatus(todo)">
                <div :class="{ 'completed': todo.isCompleted }">{{ todo.text }}</div>
                <button @click="removeTodo(todo)">Remove</button>


              </div>
              <h6> Created At : {{
        new Date(todo.createdAt).toLocaleString("en-US", options)
      }}</h6>
            </li>
          </ol>
        </div>
      </div>

    </div>
  </div>
</template>

<style>
.filter {
  font-size: 50px;
  font-weight: bold;

  font-family: Arial, Helvetica, sans-serif;
}

.dr {
  border-color: rgb(32, 10, 6);
  border: 2px solid rgb(32, 10, 6);

  width: 400px;
  height: 50px;
  font-size: medium;

}

.li {
  padding: 12px;
  display: flex;
  justify-content: space-between;

  align-items: center;
  border: 1px solid rgb(32, 10, 6);
  border-radius: 8px;
  font-family: Georgia, 'Times New Roman', Times, serif;
  margin-top: 10px;
  margin-right: 10px;
}

.container {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  flex-wrap: wrap;
  padding-top: 66px;
}

.text {
  font-weight: bolder;
}

.form-container {
  width: 500px;
  height: 500px;
}

.list-container {
  width: 500px;
  height: 500px;
}

.list {
  overflow-y: scroll;
  height: 100%;
}

.listparent {
  border: 1px solid rgb(32, 10, 6);
  border-color: rgb(32, 10, 6);
  border-radius: 20px;
  height: 501px;
  width: 500px;
  background-color: rgb(250, 247, 252);
  padding: 10px;
  scroll-margin-block: 10px;
}


.completed {
  text-decoration: line-through;
}

.new-todo {
  border-color: rgb(32, 10, 6);
  width: 400px;
  height: 50px;
  font-size: medium;
}

.Add {
  margin-top: 10px;
  width: 50px;
  height: 50px;
  border-color: rgb(32, 10, 6);
  font-weight: bold;
  font-size: medium;
}

.heading {
  font-size: 60px;
  font-weight: bold;

  font-family: Arial, Helvetica, sans-serif;
}

.todo {
  font-size: 40px;
  font-weight: bold;

  font-family: Arial, Helvetica, sans-serif;
}
</style>