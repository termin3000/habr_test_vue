<script setup>
import { reactive } from 'vue'
import MultiSelect from '@/components/select/MultiSelect.vue'

const firstSelect = reactive([])
const secondSelect = reactive([])

</script>

<template>
  <div class="page-container">
    <MultiSelect
        v-model='firstSelect'
        label="Пользователь или компания"
        placeholder="Введите логин"
        required-asterisk
        remote-url="https://habr.com/kek/v2/publication/sывuggest-mention?q="
    />
    <MultiSelect
        style="margin-top: 20px"
        v-model='secondSelect'
        label="Пользователи и/или компании"
        placeholder="Введите логин пользователя или компании"
        multi-selection
        :selection-amount="5"
        remote-url="https://habr.com/kek/v2/publication/suggest-mention?q="
    />
  </div>
</template>

<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
html, body, #App {
  height: 100%;
  width: 100%;

  font-family: "Pt Sans", serif;

  background: whitesmoke;
}
.page-container {
  padding: 20px;
  height: 100%;
}
</style>