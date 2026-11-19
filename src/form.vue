<script setup>
import { ref } from 'vue'

const input_value = ref('0')
const input_ot = ref('0')
const money = ref('0')
const total = ref('0')
const tax = ref('0')


function cal_total(){

  tax.value = input_value.value * 3/100
  money.value = input_value.value -tax.value
  total.value = money.value + input_ot.value
}
function clear_value(){
  input_value.value = 0
  input_ot.value = 0
  total.value = 0
}
</script>

<template>
  <h2>การใช้ form</h2>
  <input type="number" placeholder="กรอกข้อมูลเงินเดือน " v-model="input_value">
  <input type="number" placeholder="กรอกข้อมูล OT" v-model="input_ot">

  <br><br>
<button @click="cal_total">คำนววน</button>
<button @click="clear_value">เคลียร์คำ</button>

  <p>เงินเดือน : {{ input_value }}</p>
  <p>OT : {{ input_ot }}</p>
  <p>หัก ณ ที่จ่าย (3%) : {{ tax }}</p>
  <p>รวม : {{ total }}</p>
</template>
