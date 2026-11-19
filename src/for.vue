<script setup>
import { ref } from 'vue'

const students = ref([
  {id: 1, name:'flim',score: 67 },
  {id: 2, name:'gram',score: 67 },
  {id: 3, name:'ohm',score: 4 },
  {id: 4, name:'kunt',score: 5 }
])
</script>

<template>
  <h2>รายชื่อนักเรียน</h2>
  <ul>
    <li v-for="list in students">
      {{ list.name }} | ได้คะแนน {{ list.score }} คะแนน
    </li>

  </ul>
</template>
