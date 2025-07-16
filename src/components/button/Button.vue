<script setup>
defineProps({
  disabled: Boolean,
  onClick: Function
})
</script>

<template>
<Button :disabled="disabled" @click="onClick">Кнопка</Button>
</template>

<style scoped>

</style>