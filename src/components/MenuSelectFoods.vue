<template>
  <div class="text-center pa-2">
    <v-btn color="rgba(34, 139, 34, 0.7)" icon="mdi-plus" size="small" @click="dialog = true" />
    <v-overlay
      v-model="dialog"
      :width="width"
      class="align-center justify-center"
      @click:outside="closeMenu"
    >
      <v-card :height="height">
        <v-card-title>
          <v-row>
            <v-col>
              {{ getTargetDate() }}
            </v-col>
            <v-col cols="auto">
              <v-badge color="info" inline :content="selectedFoodsCount">
                <v-icon icon="mdi-check" size="x-large" />
              </v-badge>
            </v-col>
          </v-row>
        </v-card-title>
        <v-card-subtitle>
          {{ ration.name }}
        </v-card-subtitle>
        <v-card-text class="pa-0">
          <v-row text-align="center" justify="space-between" class="d-flex flex-nowrap pa-4">
            <v-col>
              <v-btn
                stacked
                width="100%"
                height="100%"
                density="compact"
                prepend-icon="mdi-silverware-fork-knife"
                class="py-1"
                text="Еда"
                @click="showComponent('meal')"
              />
            </v-col>
            <v-col>
              <v-btn
                stacked
                width="100%"
                height="100%"
                density="compact"
                prepend-icon="mdi-history"
                class="py-1"
                text="История"
                @click="showComponent('history')"
              />
            </v-col>
            <v-col>
              <v-btn
                stacked
                width="100%"
                height="100%"
                density="compact"
                prepend-icon="mdi-food-drumstick"
                class="py-1"
                text="Моя еда"
                @click="showComponent('myMeal')"
              />
            </v-col>
          </v-row>

          <v-card-text>
            <div v-if="showMeal">
              <MealComponent :foodBase="foodBase" @updateSelectedFood="updateSelectedFood" />
            </div>
            <div v-if="showHistory">
              <HistoryComponent @updateSelectedFood="updateSelectedFood" />
            </div>
            <div v-if="showMyMeal">
              <MyMealComponent @updateSelectedFood="updateSelectedFood" />
            </div>
          </v-card-text>
        </v-card-text>

        <template #actions>
          <v-btn text="Назад" @click="closeMenu" />
          <v-btn class="ms-auto" text="Ок" @click="addNewFood" />
        </template>
      </v-card>
    </v-overlay>
  </div>
</template>

<script setup>
import { onMounted, ref, computed } from 'vue'
import { useDisplay } from 'vuetify'
import MealComponent from './menuSelectFoods/MealComponent.vue'
import HistoryComponent from './menuSelectFoods/HistoryComponent.vue'
import MyMealComponent from './menuSelectFoods/MyMealComponent.vue'
import food_base from '../data/food_base.json'
import { useCalendarDaysStore } from '../stores/calendarDays'
import { useHistoryDataStore } from '@/stores/historyData'
import { useSelectedDataStore } from '@/stores/selectedData'

const props = defineProps({
  ration: Object
})

const { getTargetDate, addNewDataInCalendar } = useCalendarDaysStore()
const { addProductInHostory } = useHistoryDataStore()
const {
  addSelectedData,
  getSelectedData,
  clearSelectedData,
  getLengthSelectedData,
  clearLengthSelectedData
} = useSelectedDataStore()

const { name } = useDisplay()

const sizes = {
  xs: { height: 500, width: 400 },
  sm: { height: 600, width: 500 },
  md: { height: 700, width: 550 },
  lg: { height: 800, width: 650 },
  xl: { height: 900, width: 700 },
  xxl: { height: 1000, width: 800 }
}

const height = computed(() => {
  return sizes[name.value]?.height
})

const width = computed(() => {
  return sizes[name.value]?.width
})

const dialog = ref(false)
const showMeal = ref(true)
const showHistory = ref(false)
const showMyMeal = ref(false)
const selectedFoodsCount = ref(0)
const selectedData = ref([])
const foodBase = ref([])

function showComponent(nameComponent) {
  switch (nameComponent) {
    case 'meal': {
      showMeal.value = true
      showHistory.value = false
      showMyMeal.value = false
      break
    }
    case 'history': {
      showMeal.value = false
      showHistory.value = true
      showMyMeal.value = false
      break
    }
    case 'myMeal': {
      showMeal.value = false
      showHistory.value = false
      showMyMeal.value = true
      break
    }
  }
}

function updateSelectedFood(selectedFood, nameComponent) {
  selectedData.value = selectedFood
  addSelectedData(selectedFood, nameComponent)
  selectedFoodsCount.value = getLengthSelectedData()
}

function addNewFood() {
  addNewDataInCalendar({
    date: getTargetDate(),
    ration: props.ration,
    selectedFoods: getSelectedData()
  })
  addProductInHostory(selectedData.value)

  resetMenuSelectFoods()
  dialog.value = false
}

function closeMenu() {
  resetMenuSelectFoods()
  dialog.value = false
}

function resetMenuSelectFoods() {
  clearSelectedData()
  clearLengthSelectedData()
  localStorage.removeItem('filteredFood')
  localStorage.removeItem('selectedFoodForMeal')
  selectedFoodsCount.value = 0
}

onMounted(() => {
  foodBase.value = food_base
  resetMenuSelectFoods()
  selectedFoodsCount.value = 0
})
</script>
