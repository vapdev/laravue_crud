<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const newTask = ref('')
const tasks = ref([])

onMounted(() => {
  getTasks()
})

async function deleteTask (task) {
  const index = tasks.value.indexOf(task)
  tasks.value.splice(index, 1)
  axios.delete('http://127.0.0.1:8000/api/task/' + task.id, {
    headers: {
      'Content-Type': 'application/json'
    }
  })
  .catch((error) => {
    console.log(error)
  })
}

async function getTasks () {
  axios.get('http://127.0.0.1:8000/api/tasks')
  .then(response => {
    tasks.value = response.data
    for (let i = 0; i < tasks.value.length; i++) {
      if (tasks.value[i].checked === 0) {
        tasks.value[i].checked = false
      } else tasks.value[i].checked = true
    }
  })
  .catch((error) => {
    console.log(error)
  })
}

async function toggleFinish (task) {
  axios.put('http://127.0.0.1:8000/api/task/' + task.id)
  .catch((error) => {
    console.log(error)
  })
}


async function addTask () {
  const task = {
    title: newTask.value,
    checked: false
  }
  await axios.post('http://127.0.0.1:8000/api/tasks', task, {
    headers: {
      'Content-Type': 'application/json'
    }
  }).catch((error) => {
    console.log(error)
  })
  newTask.value = ''
  getTasks()
}

</script>

<template>
  <q-page>
    <q-input v-model="newTask" filled bottom-slots placeholder="Add new task" @keyup.enter="addTask">
      <template v-slot:append>
        <q-btn @click="addTask()" round dense flat icon="add" />
      </template>
    </q-input>
    <q-list class="bg-grey-4" separator bordered>
      <q-item v-for="task in tasks" :key="task.title" @click="task.checked=!task.checked" :class="{'checked': task.checked}" v-ripple>
        <q-item-section avatar>
          <q-checkbox @click="toggleFinish(task)" v-model="task.checked"/>
        </q-item-section>
        <q-item-section>
          <q-item-label>{{task.title}}</q-item-label>
        </q-item-section>
        <q-item-section side>
          <q-btn @click.stop="deleteTask(task)" flat color="secondary" icon="delete" />
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>