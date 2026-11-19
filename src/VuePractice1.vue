<script setup>
import { ref } from 'vue'
import { useStorage } from '@vueuse/core'

const data_list = useStorage('data_list', [])
const title = ref('')
const amount = ref('')
const errorMessage = ref('')

// กฎสำหรับตรวจสอบข้อมูล (Validation Rules)
const titleRules = [
  (v) => (!!v && v.trim().length > 0) || 'กรุณากรอกชื่อรายการ',
]

const amountRules = [
  (v) => (v !== '' && v !== null && v !== undefined) || 'กรุณากรอกจำนวนเงิน',
  (v) => Number(v) > 0 || 'จำนวนเงินต้องมากกว่า 0',
]

// สถานะสำหรับ Dialog แก้ไขรายการ
const editDialog = ref(false)
const editingIndex = ref(null)
const editForm = ref({
  title: '',
  amount: '',
  type: 'income',
})
const editError = ref('')

function add(type) {
  errorMessage.value = ''

  if (!title.value.trim() || amount.value === '' || Number(amount.value) <= 0) {
    errorMessage.value = 'กรุณากรอกชื่อรายการและจำนวนเงิน (มากกว่า 0) ให้ครบถ้วน'
    return
  }

  data_list.value.push({
    title: title.value.trim(),
    amount: Number(amount.value),
    type: type,
  })

  title.value = ''
  amount.value = ''
}

function openEdit(index) {
  const item = data_list.value[index]
  editingIndex.value = index
  editForm.value = {
    title: item.title,
    amount: item.amount,
    type: item.type,
  }
  editError.value = ''
  editDialog.value = true
}

function saveEdit() {
  editError.value = ''

  if (!editForm.value.title.trim() || editForm.value.amount === '' || Number(editForm.value.amount) <= 0) {
    editError.value = 'กรุณากรอกชื่อรายการและจำนวนเงิน (มากกว่า 0) ให้ครบถ้วน'
    return
  }

  if (editingIndex.value !== null) {
    data_list.value[editingIndex.value] = {
      title: editForm.value.title.trim(),
      amount: Number(editForm.value.amount),
      type: editForm.value.type,
    }
  }

  editDialog.value = false
}

function remove(index) {
  data_list.value.splice(index, 1)
}

function getBalance() {
  let total = 0
  for (const item of data_list.value) {
    if (item.type == 'income') {
      total += item.amount
    } else {
      total -= item.amount
    }
  }
  return total
}
</script>

<template>
  <v-container style="max-width: 540px;" class="text-center">
    <h1>รายรับ-รายจ่าย</h1>
    <h2 class="mb-4">คงเหลือ {{ getBalance() }} บาท</h2>

    <v-alert
      v-if="errorMessage"
      type="error"
      variant="tonal"
      density="compact"
      class="mb-3 text-start"
      closable
      @click:close="errorMessage = ''"
    >
      {{ errorMessage }}
    </v-alert>

    <v-text-field
      v-model="title"
      label="รายการ"
      :rules="titleRules"
      class="mb-2"
    />
    <v-text-field
      v-model="amount"
      label="จำนวนเงิน"
      type="number"
      min="0"
      :rules="amountRules"
      class="mb-3"
    />

    <v-btn color="green" class="mr-2" @click="add('income')">+ รายรับ</v-btn>
    <v-btn color="red" @click="add('expense')">- รายจ่าย</v-btn>

    <br /><br />

    <v-list>
      <v-list-item v-for="(item, index) in data_list" :key="index">
        <div class="d-flex align-center">
          <span>{{ item.title }}</span>
          <v-spacer />
          <span :class="item.type == 'income' ? 'text-green' : 'text-red'" class="mr-3">
            {{ item.type == 'income' ? '+' : '-' }}{{ item.amount }}
          </span>
          <v-btn size="small" color="amber-darken-2" variant="tonal" class="mr-1" @click="openEdit(index)">
            แก้ไข
          </v-btn>
          <v-btn size="small" color="red" @click="remove(index)">ลบ</v-btn>
        </div>
      </v-list-item>
    </v-list>

    <!-- Dialog สำหรับแก้ไขรายการ -->
    <v-dialog v-model="editDialog" max-width="420px">
      <v-card>
        <v-card-title>แก้ไขรายการ</v-card-title>
        <v-card-text>
          <v-alert
            v-if="editError"
            type="error"
            variant="tonal"
            density="compact"
            class="mb-3"
          >
            {{ editError }}
          </v-alert>

          <v-text-field
            v-model="editForm.title"
            label="รายการ"
            :rules="titleRules"
            class="mb-2"
          />
          <v-text-field
            v-model="editForm.amount"
            label="จำนวนเงิน"
            type="number"
            min="0"
            :rules="amountRules"
            class="mb-2"
          />
          <v-select
            v-model="editForm.type"
            label="ประเภทรายการ"
            :items="[
              { title: 'รายรับ (+)', value: 'income' },
              { title: 'รายจ่าย (-)', value: 'expense' },
            ]"
            item-title="title"
            item-value="value"
          />
        </v-card-text>
        <v-card-actions class="justify-end">
          <v-btn variant="text" @click="editDialog = false">ยกเลิก</v-btn>
          <v-btn color="primary" variant="flat" @click="saveEdit">บันทึก</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
