<script setup>
import { ref, reactive } from 'vue'

const loginStatus = ref(false)
const count = ref(0)
const detail = reactive({
  name : 'แกรม',
  cash : 50
})
function addcash(){
  detail.cash+=50
}
const student = {
  name :'แกรม',
  department :'เทคโนโลยีสารสนเทศ',
}

</script>


<template>
<button @click="loginStatus = true">Login</button>
<button @click="loginStatus = false">Logout</button>
<p>{{ loginStatus }}</p>
<p v-if="loginStatus==true">login</p>
<p v-else>logout</p>

<div v-if="loginStatus==true">
  <p>{{ detail.name  }} มีเงิน {{ detail.cash }} บาท</p>
  <button @click="addcash">เพิ่มเงิน 50 บาท</button>
  <button @click="detail.cash-=50">ลดเงิน 50 บาท</button>

  <p>จำนวน : {{ count }}</p>
  <button @click="count++">เพิ่มจำนวน</button>
  <button @click="count--">ลดจำนวน</button>

  <h1>ข้อมูลนักเรียน</h1>
  <p>ชื่อ : {{ student.name }}</p>
  <p>สาขา : {{ student.department }}</p>

  </div>
</template>
