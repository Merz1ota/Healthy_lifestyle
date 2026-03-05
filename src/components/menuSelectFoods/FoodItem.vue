<template>
  <v-list lines="one" :height="height">
    <div v-if="loader" class="text-center">
      <v-progress-circular indeterminate />
    </div>
    <v-row
      no-gutters
      v-for="food in props.filteredFood"
      :key="food.id"
      class="hover-color"
      @click="openInfoProduct(food)"
    >
      <v-col>
        <v-list-item>
          <v-list-item-title> {{ food.name }} </v-list-item-title>
          <v-list-item-subtitle> {{ food.calories }} </v-list-item-subtitle>
        </v-list-item>
      </v-col>
      <v-col cols="auto">
        <v-checkbox v-model="selected" :value="food" @click.stop />
      </v-col>
    </v-row>

    <v-dialog max-width="800" v-model="dialog">
      <v-card :title="dataProductForSettings.name">
        <v-card-item>
          <v-text-field
            prepend-icon="mdi-plus-minus-variant"
            type="number"
            v-model="productWeight"
            @update:modelValue="updateDataForProduct"
          />
        </v-card-item>

        <v-row no-gutters class="text-center">
          <v-col class="ma-1 bg-grey">
            <v-card-item>
              <v-card-title> Калории </v-card-title>
              <v-card-text class="size-text">
                {{ dataProductForSettings.calories }}
              </v-card-text>
            </v-card-item>
          </v-col>
          <v-col class="ma-1 bg-grey">
            <v-card-item>
              <v-card-title> Белок </v-card-title>
              <v-card-text class="size-text">
                {{ dataProductForSettings.proteins }}
              </v-card-text>
            </v-card-item>
          </v-col>
        </v-row>

        <v-row no-gutters class="text-center">
          <v-col class="ma-1 bg-grey">
            <v-card-item>
              <v-card-title> Жиры </v-card-title>
              <v-card-text class="size-text">
                {{ dataProductForSettings.fats }}
              </v-card-text>
            </v-card-item>
          </v-col>
          <v-col class="ma-1 bg-grey">
            <v-card-item>
              <v-card-title> Углеводы </v-card-title>
              <v-card-text class="size-text">
                {{ dataProductForSettings.carbs }}
              </v-card-text>
            </v-card-item>
          </v-col>
        </v-row>

        <v-card-actions>
          <v-spacer />
          <v-btn text="Close" @click="closeSettings" />
          <v-btn text="Save" @click="saveSettings" />
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-list>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import trimString from '@/composables/trimString.js'
import { useDisplay } from 'vuetify'

const { name } = useDisplay()

const dialog = ref(false)
const selected = ref([])
const dataProductForSettings = ref({})
const productWeight = ref(100)

const props = defineProps({
  filteredFood: Array,
  loader: Boolean
})

const height = computed(() => {
  switch (name.value) {
    case 'xs':
      return 150
    case 'sm':
      return 270
    case 'md':
      return 370
    case 'lg':
      return 470
    case 'xl':
      return 570
    case 'xxl':
      return 670
    default:
      return 380
  }
})

const emit = defineEmits(['updateSelectedFood'])

function openInfoProduct(food) {
  dialog.value = true
  dataProductForSettings.value = Object.assign({}, food)
  if (dataProductForSettings.value.productWeight) {
    productWeight.value = dataProductForSettings.value.productWeight
  } else {
    productWeight.value = 100
  }
}

function updateDataForProduct(newData) {
  let inputValue = newData > 0 ? newData : 100
  props.filteredFood.forEach((el) => {
    if (el.id == dataProductForSettings.value.id) {
      dataProductForSettings.value.calories = `${((+trimString(el.calories) * inputValue) / el.productWeight).toFixed(1).replace(/\.0$/, '')} кКал`
      dataProductForSettings.value.proteins = `${((+trimString(el.proteins) * inputValue) / el.productWeight).toFixed(1).replace(/\.0$/, '')} г`
      dataProductForSettings.value.fats = `${((+trimString(el.fats) * inputValue) / el.productWeight).toFixed(1).replace(/\.0$/, '')} г`
      dataProductForSettings.value.carbs = `${((+trimString(el.carbs) * inputValue) / el.productWeight).toFixed(1).replace(/\.0$/, '')} г`
    }
  })
}

function saveSettings() {
  props.filteredFood.forEach((el) => {
    if (el.id == dataProductForSettings.value.id) {
      el.calories = dataProductForSettings.value.calories
      el.proteins = dataProductForSettings.value.proteins
      el.fats = dataProductForSettings.value.fats
      el.carbs = dataProductForSettings.value.carbs
      el.productWeight = +productWeight.value
    }
    setSelectedItems()
  })
  dataProductForSettings.value = {}
  productWeight.value = 100
  dialog.value = false
}

function closeSettings() {
  dataProductForSettings.value = {}
  productWeight.value = 100
  dialog.value = false
}

function setSelectedItems() {
  if (selected.value) {
    selected.value.forEach((item) => {
      const foundFood = props.filteredFood.find((food) => food.name === item.name)
      if (foundFood) {
        foundFood.calories = item.calories
        foundFood.proteins = item.proteins
        foundFood.fats = item.fats
        foundFood.carbs = item.carbs
        foundFood.productWeight = item.productWeight
      }
    })
  }
}

watch(selected, () => {
  localStorage.setItem('selectedFoodForMeal', JSON.stringify(selected.value))
  emit('updateSelectedFood', selected.value)
})

onMounted(() => {
  if (localStorage.getItem('selectedFoodForMeal')) {
    selected.value = JSON.parse(localStorage.getItem('selectedFoodForMeal'))
    setSelectedItems()
  }
})
</script>

<style scoped lang="scss">
.hover-color:hover {
  background-color: lightgray;
}
.size-text {
  font-size: 18px;
}
</style>
