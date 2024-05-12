<script setup>
import { ref, reactive, watch, computed, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => [],
    required: true
  },
  remoteUrl: {
    type: String,
    default: null,
    required: true
  },
  multiSelection: {
    type: Boolean,
    default: false
  },
  selectionAmount: {
    type: Number,
    default: 1
  },
  label: {
    type: String,
    default: null
  },
  requiredAsterisk: {
    type: Boolean,
    default: false
  },
  placeholder: {
    type: String,
    default: ''
  }
})
const emits = defineEmits(['update:modelValue'])

const selectWrapper = ref(null)

const dropdown = ref(null)

const text = ref('')
const dropdownData = reactive([])
const dropdownState = ref(false)
const locker = ref('')
const loader = ref(false)

const showInput = computed(() => {
  return props.modelValue.length === 0 && !props.multiSelection ||
         props.multiSelection && props.modelValue.length !== props.selectionAmount
})

const avatar = computed(() => (item) => {
  if (item.avatar) return item.avatar

  return item.type === 'user' ? new URL('./assets/user.png', import.meta.url) : new URL('./assets/company.png', import.meta.url)
})
const name = computed(() => (item) => {
  return item.name ? item.name : item.alias
})
const alias = computed(() => (item) => {
  return item.type === 'company' ? 'Компания' : `@${item.alias}`
})

const dropdownStyle = computed(() => {
  const style = {
    top: `${selectWrapper.value.offsetHeight + 10}px`
  }

  if (dropdownData.length > 4) {
    style.height = `200px`
  }

  return style
})

watch(text, async () => {
  switch (true) {
    case text.value.length < 3 && !props.multiSelection ||
         props.modelValue.length === props.selectionAmount:
      closeDropdown()
      dropdownData.splice(0)
      break
    case text.value.length < 3 && props.multiSelection &&
         props.modelValue.length !== props.selectionAmount:
      break
    default:
      await request()
  }
})
watch(() => props.modelValue, (val) => {
  if (
      val.length === 1 && !props.multiSelection ||
      props.multiSelection && val.length === props.selectionAmount
  ) closeDropdown()
}, { deep: true})

function openDropdown() {
  if (text.value.length >= 3) dropdownState.value = true
}
function closeDropdown() {
  dropdownState.value = false
}
function clickOutside(e) {
  if (e.keyCode === 9 || e.type === 'click') {
    if (dropdownState.value && e.target.closest('.select-wrapper') !== selectWrapper.value) {
      closeDropdown()
    }
  }
}

function selectItem(item) {
  if (props.modelValue.includes(item.alias)) return
  const arr = props.modelValue.slice(0)
  arr.push(item.alias)
  emits('update:modelValue', props.modelValue.splice(0, props.modelValue.length, ...arr))
  text.value = ''
}
function deleteItem(item) {
  const index = props.modelValue.findIndex(elem => elem === item)
  emits('update:modelValue', props.modelValue.splice(index, 1))
}

function debounce(func, ms) {
  let timeout;
  return function() {
    clearTimeout(timeout)
    timeout = setTimeout(async () => {func.apply(this, arguments)}, ms)
  }
}
async function requestData() {
  try {
    loader.value = true
    const resp = await fetch(props.remoteUrl + text.value)
    const data = await resp.json()

    dropdownData.splice(0)
    if (data.data.length) {
      data.data.forEach(item => dropdownData.push(item))
    } else {
      locker.value = 'Нет совпадений.'
    }
    loader.value = false
    openDropdown()
  } catch(e) {
    locker.value = 'Ошибка выполнения запроса'
    loader.value = false
    openDropdown()
  }
}
const request = debounce(requestData, 400)

onMounted(() => {
  document.addEventListener('click', clickOutside)
  document.addEventListener("keyup", clickOutside)
})
onUnmounted(() => {
  document.removeEventListener('click', clickOutside)
  document.removeEventListener('keyup', clickOutside)
})

</script>

<template>
  <div
      ref="selectWrapper"
      class="select-wrapper"
  >
    <div
        v-if="label"
        class="input-label"
    >
      <img
          v-if="requiredAsterisk"
          class="required-icon"
          src="./assets/required.png"
          alt=""
      />
      <span>{{label}}</span>
    </div>
    <div
        class="input-area"
    >
      <div
          v-for="item in modelValue"
          :key="item"
          class="tag"
      >
        <span class="tag-description">@{{item}}</span>
        <img
            class="tag-cross"
            src="./assets/close.png"
            alt=""
            tabindex="0"
            @click="deleteItem(item)"
            @keydown.enter="deleteItem(item)"
        />
      </div>
      <input
          v-model="text"
          v-show="showInput"
          :placeholder="modelValue.length ? '' : placeholder"
          @focus="openDropdown"
      />
    </div>
    <div
        ref="dropdown"
        v-if="dropdownState"
        class="dropdown"
        :style="dropdownStyle"
    >
      <div v-if="loader" class="loader" />
      <template v-if="dropdownData.length">
        <div
            v-for="item in dropdownData"
            :key="item.alias"
            class="dropdown-item"
            tabindex="0"
            @click="selectItem(item)"
            @keydown.enter="selectItem(item)"
        >
          <img class="avatar" :src="avatar(item)" alt="">
          <div class="dropdown-item-description">
          <span class="name">
            {{name(item)}}
          </span>
            <span class="alias">
            {{alias(item)}}
          </span>
          </div>
        </div>
      </template>
      <div v-else class="locker">
        <span class="locker-text">{{locker}}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.select-wrapper {
  position: relative;
  width: 100%;
}

.input-label {
  display: flex;

  margin-bottom: 3px;

  width: 100%;

  line-height: 1;
}
.required-icon {
  margin-right: 5px;

  width: 7px;
  height: 7px;
}

.input-area {
  display: flex;
  align-items: center;
  flex-wrap: wrap;

  padding: 2px 3px;

  min-height: 32px;

  background: white;
  border: 1px solid #d6dfdf;
  border-radius: 3px;
}
.input-area input {
  flex-grow: 1;
  border: none;
}
.input-area input:focus {
  outline: none;
}

.tag {
  display: flex;
  align-items: center;

  height: 100%;

  margin:  3px;
  padding: 2px 3px;

  line-height: 1;

  border-radius: 3px;
  background: #91a5b9;
}
.tag-description {
  cursor: default;
  user-select: none;

  color: white;
}
.tag-cross {
  margin-left: 5px;
  width: 10px;
  height: 10px;
  cursor: pointer;
}
.tag-cross:focus {
  outline: 1px solid white;
  border-radius: 2px;
}

.dropdown {
  overflow: auto;
  position: absolute;

  width: 100%;
  max-width: 300px;

  background: white;
  border: 1px solid #d6dfdf;
  z-index: 5;
}
.dropdown::-webkit-scrollbar {
   height: 8px;
   width: 8px;
   background: transparent;
 }
.dropdown::-webkit-scrollbar-track {
   background: transparent;
 }
.dropdown::-webkit-scrollbar-thumb {
   border-radius: 20px;
   background: #d6dfdf;
 }

.dropdown-item, .locker {
  user-select: none;

  display: flex;
  align-items: center;
  padding: 5px 10px;
}
.dropdown-item:hover {
  cursor: pointer;
  background: #d6dfdf;
}
.dropdown-item:focus {
  cursor: pointer;

  background: #d6dfdf;
  outline: none;
}
.dropdown-item-description {
  display: flex;
  flex-direction: column;
}
.dropdown-item .avatar {
  margin-right: 5px;
  width: 35px;
  height: 100%;
}
.dropdown-item .name {
  font-weight: bold;
}
.dropdown-item .alias, .locker {
  color: #949494;
}

.locker-text {
  width: 100%;
  text-align: center;
}
.loader {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, .1)
}
</style>