<template>
  <q-page padding>
    <div class="q-pa-md" style="max-width: 600px; margin: auto">

      <!-- Form Area -->
      <div class="row q-col-gutter-md items-center">
        <div class="col">
          <q-input 
            v-model="newTask.title" 
            label="Task Name" 
            outlined 
            dense 
            clearable
          />
        </div>
        <div class="col-auto">
          <q-btn 
            label="Add" 
            color="primary" 
            @click="addTask" 
            icon="add" 
          />
        </div>
      </div>

      

      <!-- Task Cards -->
      <div class="q-mt-lg">
        <q-card
          v-for="task in tasks"
          :key="task.id"
          class="q-mb-md"
        >
          <q-card-section class="row items-center justify-between">
            <q-input 
              v-model="task.title" 
              dense 
              borderless 
              style="flex: 1;"
            />
            <div class="q-gutter-sm">
              <q-btn 
                flat 
                dense 
                icon="save" 
                color="green" 
                @click="updateTask(task)" 
              />
              <q-btn 
                flat 
                dense 
                icon="delete" 
                color="red" 
                @click="deleteTask(task.id)" 
              />
            </div>
          </q-card-section>
        </q-card>
      </div>


    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API_URL = 'https://jsonplaceholder.typicode.com/todos'

const tasks = ref([])
const newTask = ref({ title: '', completed: false, userId: 1 })

const fetchTasks = async () => {
  try {
    const res = await axios.get(`${API_URL}?_limit=50`)
    tasks.value = res.data
  } catch (error) {
    console.error('Error fetching tasks:', error)
  }
}

const addTask = async () => {
  if (!newTask.value.title.trim()) return
  try {
    const res = await axios.post(API_URL, {
      title: newTask.value.title,
      completed: false,
      userId: 1
    })
    tasks.value.unshift(res.data)
    newTask.value.title = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const updateTask = async (task) => {
  const index = tasks.value.findIndex((t) => t.id === task.id);
  if (index !== -1) {
    tasks.value[index].title = task.title;
  }

  try {
    await axios.put(`${API_URL}/${task.id}`, {
      ...task,
    });
    console.log('Task updated:', task.id);
  } catch (error) {
    console.error('Error updating task:', error);

  }
};

const deleteTask = async (id) => {
  try {
    await axios.delete(`${API_URL}/${id}`)
    tasks.value = tasks.value.filter(task => task.id !== id)
  } catch (error) {
    console.error('Delete failed, removing locally anyway:', error)
    tasks.value = tasks.value.filter(task => task.id !== id)
  }
}

onMounted(fetchTasks)
</script>