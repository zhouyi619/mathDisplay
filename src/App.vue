<script setup>
import Main from './components/Main.vue'
import { NNotificationProvider } from 'naive-ui'
</script>

<template>
  <div>
  <NNotificationProvider>
    <Main id="main"/>
  </NNotificationProvider>
  </div>
</template>

<style scoped>
</style>
