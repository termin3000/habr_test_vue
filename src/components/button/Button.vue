<script setup>
defineProps({
  disabled: Boolean
})
</script>

<template>
<Button :disabled="disabled">Кнопка</Button>
</template>

<style scoped>

</style>