<script setup>
import RecordSample from './components/RecordSample.vue'
</script>

<template>
  <RecordSample 
    recordStatus="NONE"
    :togglePauseRecording="() => {}"
    :toggleRecording="() => {}"
  />
</template>

<style>
/* Add your custom styles here */
.margin-small { margin: 8px; }
.margin-x-small { margin: 0 8px; }
.padding-extra-small { padding: 4px; }
.bg-grey-4 { background-color: #bdbdbd; }
.no-stroke { border: none; }
.bg-dark { background-color: #333; }
.text-white { color: white; }
.bg-white { background-color: white; }
.text-dark { color: #333; }
</style>

