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
          bordered
        >
          
          <q-banner 
            v-if="task._updated" 
            class="bg-green-2 text-green-10" 
            dense 
            rounded
          >
            Task updated successfully!
          </q-banner>

          <q-card-section>
            <q-input 
              v-model="task.title" 
              label="Task Title"
              outlined 
              dense 
              clearable
            />
          </q-card-section>

          <q-card-actions align="right">
            <q-btn 
              color="primary" 
              label="Update Task" 
              @click="updateTask(task)"
              icon="save"
            />
            <q-btn 
              color="red" 
              flat 
              label="Delete" 
              @click="deleteTask(task.id)" 
              icon="delete"
            />
          </q-card-actions>
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
    const res = await axios.get(`${API_URL}?_limit=10`) 
    tasks.value = res.data.map(task => ({ ...task, _updated: false }))
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
    tasks.value.unshift({ ...res.data, _updated: false })
    newTask.value.title = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const updateTask = async (task) => {
  const index = tasks.value.findIndex((t) => t.id === task.id)
  if (index === -1) return

  try {
    await axios.put(`${API_URL}/${task.id}`, { ...task })
    tasks.value[index]._updated = true

    setTimeout(() => {
      tasks.value[index]._updated = false
    }, 2000)
  } catch (error) {
    console.error('Error updating task:', error)
  }
}

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
