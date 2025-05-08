<template>
  <q-page padding>
    <div class="q-pa-md" style="max-width: 600px; margin: auto">

      <div class="row q-col-gutter-md items-center">
        <div class="col">
          <q-input
            v-model="newTask.title"
            label="Task Name"
            outlined
            dense
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

          <q-card-section v-if="!task._isEditing">
            <div class="text-h6">{{ task.title }}</div>
          </q-card-section>

          <q-card-section v-else>
            <q-input
              v-model="task.title"
              label="Edit Task Title"
              outlined
              dense
            />
          </q-card-section>

          <q-card-actions align="right">
            <q-btn
              v-if="!task._isEditing"
              color="primary"
              label="Update Task"
              @click="startEdit(task)"
              icon="edit"
            />
            <template v-else>
              <q-btn
                color="primary"
                label="Save"
                @click="updateTask(task)"
                icon="save"
              />
              <q-btn
                color="grey"
                flat
                label="Cancel"
                @click="cancelEdit(task)"
                icon="cancel"
              />
            </template>
            <q-btn
              v-if="!task._isEditing"
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
let idIndex = 0;

const fetchTasks = async () => {
  try {
    const res = await axios.get(`${API_URL}?_limit=50`)
    tasks.value = res.data.map(task => ({ ...task, _updated: false, _isEditing: false, _isClientAdded: false }))
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
    res.data.id += idIndex
    idIndex+=1
    tasks.value.unshift({ ...res.data, _updated: false, _isEditing: false, _isClientAdded: true }) // Mark as client-added
    newTask.value.title = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const startEdit = (task) => {
  task._originalTitle = task.title
  task._isEditing = true
}

const cancelEdit = (task) => {
  task.title = task._originalTitle
  task._isEditing = false
}

const updateTask = async (task) => {
  const index = tasks.value.findIndex((t) => t.id === task.id)
  if (index === -1) return

  if (!task._isClientAdded) {
    try {
      await axios.put(`${API_URL}/${task.id}`, { ...task })
      tasks.value[index]._updated = true
      task._isEditing = false
      setTimeout(() => {
        tasks.value[index]._updated = false
      }, 2000)
    } catch (error) {
      console.error('Error updating task:', error)
    }
  } else {
    tasks.value[index].title = task.title;
    tasks.value[index]._updated = true;
    task._isEditing = false;
    setTimeout(() => {
      tasks.value[index]._updated = false;
    }, 2000);

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