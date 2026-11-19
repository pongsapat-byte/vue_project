<script setup>
import { ref } from 'vue';

const task = ref('')
const task_list = ref([])
const alert_status = ref(false)

function add_task(){
  alert_status.value = false
  if(task.value ==""){
    alert_status.value = true
  }else{
    task_list.value.push(task.value)
    task.value = ""

  }
}
function del_task(index){
  task_list.value.slice(index, 1)
}

</script>

<template>
  <v-container style="max-width: 500px;" class="text-center">

    <h1>To do list</h1>

    <v-alert  v-if="alert_status"
      text="ใส่สิ่งที่ต้องการจะทำ"
      type="worning"
    ></v-alert>
    <br>
    <v-text-field
      label="สิ่งที่ต้องทำ"
      prepend-icon="mdi-list-box-outline"
      v-model="task"
    ></v-text-field>
    <v-btn color="blue" prepend-icon="mdi-plus" @click="add_task">
      เพิ่มรายการ
    </v-btn>
    <ol class="text-left">
      <li v-for="(list, index) in task_list" :key="index">
        <h3>{{ list }} <v-btn color="red" size="small" @click="del_task(index)">ลบ</v-btn></h3>
      </li>
    </ol>
  </v-container>

</template>
