<script setup>
  import {ref} from 'vue'
  const user = ref('')
  const password = ref('')
  const result = ref('')

function check_data(){
  if(user.value == 'admin' && password.value == '1234'){
    result.value = 'เข้าสู่ระบบสำเร็จ'
  }else{
    result.value = 'เข้าสู่ระบบไม่สำเร็จ'
  }
}
function clear_data(){
  user.value = ''
  password.value = ''
  result.value = ''
}
</script>




<template>
  <v-container style="max-width: 400px;">
    <h1>login </h1>

    <v-text-field v-model="user" lable="username" prepend-icon="mdi-account"></v-text-field>
    <v-text-field v-model="password " lable="password" type="password" prepend-icon="mdi-lock"></v-text-field>

    <v-btn color="blue" prepend-icon="mdi-check-circle" class="mr-4" @click="check_data">
      เข้าสู่ระบบ
    </v-btn>
    <v-btn color="red" prepend-icon="mdi-close-circle" @click="clear_data">
      ยกเลิก
    </v-btn>
    <p>{{ result }}</p>
  </v-container>
</template>
