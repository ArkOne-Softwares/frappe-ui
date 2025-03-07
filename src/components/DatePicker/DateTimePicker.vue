<template>
  <Popover
    @open="selectCurrentMonthYear"
    class="flex w-full [&>div:first-child]:w-full"
    :placement="placement"
  >
    <template #target="{ togglePopover }">
      <TextInput
        readonly
        type="text"
        icon-left="calendar"
        :placeholder="placeholder"
        :value="dateValue && formatter ? formatter(dateValue) : dateValue"
        @focus="!readonly ? togglePopover() : null"
        class="w-full"
        :class="inputClass"
        v-bind="$attrs"
      />
    </template>

    <template #body="{ togglePopover }">
      <div
        class="w-fit select-none text-base text-ink-gray-9 divide-y divide-outline-gray-modals rounded-lg bg-surface-modal shadow-2xl ring-1 ring-black ring-opacity-5 focus:outline-none"
        :class="marginClass"
      >
        <!-- Month Switcher -->
        <div class="flex items-center p-1 text-ink-gray-4">
          <Button variant="ghost" class="h-7 w-7" @click="prevMonth">
            <FeatherIcon
              :stroke-width="2"
              name="chevron-left"
              class="h-4 w-4"
            />
          </Button>
          <div class="flex-1 text-center text-base font-medium text-ink-gray-6">
            {{ formattedMonth }}
          </div>
          <Button variant="ghost" class="h-7 w-7" @click="nextMonth">
            <FeatherIcon
              :stroke-width="2"
              name="chevron-right"
              class="h-4 w-4"
            />
          </Button>
        </div>

        <!-- Date Time Input -->
        <div class="flex items-center justify-center gap-1 p-1">
          <TextInput
            class="text-sm"
            type="text"
            :value="dateValue"
            @change="(e: Event) => { updateDate((e.target as HTMLInputElement).value); }"
          />
          <Button
            :label="'Now'"
            class="text-sm"
            @click="() => { selectDate(getDate(), true); }"
          />
        </div>

        <!-- Date Picker -->
        <div class="flex flex-col items-center justify-center p-1 text-ink-gray-8">
          <div class="flex items-center text-xs uppercase">
            <div
              class="flex h-6 w-8 items-center justify-center text-center"
              v-for="(d, i) in ['s', 'm', 't', 'w', 't', 'f', 's']"
              :key="i"
            >
              {{ d }}
            </div>
          </div>
          <div
            class="flex items-center"
            v-for="(week, i) in datesAsWeeks"
            :key="i"
          >
            <div
              v-for="date in week"
              :key="toValue(date)"
              class="flex h-8 w-8 cursor-pointer items-center justify-center rounded hover:bg-surface-gray-2"
              :class="{
                'text-ink-gray-3': date.getMonth() !== currentMonth - 1,
                'font-extrabold text-ink-gray-9': toValue(date) === toValue(today),
                'bg-surface-gray-6 text-ink-white hover:bg-surface-gray-6': toValue(date) === dateValue,
              }"
              @click="() => { selectDate(date);}"
            >
              {{ date.getDate() }}
            </div>
          </div>
        </div>

        <!-- New Time Picker -->
        <div class="flex justify-around gap-4 p-2">
          <div class="text-center">
            <div class="text-sm font-bold">Hour</div>
            <Button variant="ghost" class="m-1" @click="increaseHour">
              <FeatherIcon name="chevron-up" class="h-4 w-4" />
            </Button>
            <div class="text-xl font-bold">{{ twoDigit(hour) }}</div>
            <Button variant="ghost" class="m-1" @click="decreaseHour">
              <FeatherIcon name="chevron-down" class="h-4 w-4" />
            </Button>
          </div>
          <div class="text-center">
            <div class="text-sm font-bold">Minute</div>
            <Button variant="ghost" class="m-1" @click="increaseMinute">
              <FeatherIcon name="chevron-up" class="h-4 w-4" />
            </Button>
            <div class="text-xl font-bold">{{ twoDigit(minute) }}</div>
            <Button variant="ghost" class="m-1" @click="decreaseMinute">
              <FeatherIcon name="chevron-down" class="h-4 w-4" />
            </Button>
          </div>
          <div class="text-center">
            <div class="text-sm font-bold">Second</div>
            <Button variant="ghost" class="m-1" @click="increaseSecond">
              <FeatherIcon name="chevron-up" class="h-4 w-4" />
            </Button>
            <div class="text-xl font-bold">{{ twoDigit(second) }}</div>
            <Button variant="ghost" class="m-1" @click="decreaseSecond">
              <FeatherIcon name="chevron-down" class="h-4 w-4" />
            </Button>
            </div>
        </div>

        <!-- Actions -->
        <div class="flex justify-between p-1">
          <Button
            :label="'Clear'"
            class="text-sm"
            theme="red"
            @click="() => { selectDate(''); togglePopover() }"
          />
          <Button
            :label="'Done'"
            class="text-sm"
            @click="togglePopover"
          />
        </div>
      </div>
    </template>
  </Popover>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

import { Button } from '../Button'
import Popover from '../Popover.vue'
import FeatherIcon from '../FeatherIcon.vue'
import TextInput from '../TextInput.vue'

import { getDate } from './utils'
import { useDatePicker } from './useDatePicker'
import { dayjs, dayjsLocal, dayjsSystem } from '../../utils/dayjs'

import type { DatePickerEmits, DatePickerProps } from './DatePicker'

const props = defineProps<DatePickerProps>()
const emit = defineEmits<DatePickerEmits>()

const {
  currentYear,
  currentMonth,
  today,
  datesAsWeeks,
  formattedMonth,
  prevMonth,
  nextMonth,
} = useDatePicker()

const marginClass = computed(() => {
  let _marginClass = 'mt-2'
  if (props.placement?.startsWith('top')) {
    _marginClass = 'mb-2'
  } else if (props.placement?.startsWith('left')) {
    _marginClass = 'mr-2'
  } else if (props.placement?.startsWith('right')) {
    _marginClass = 'ml-2'
  }
  return _marginClass
})

const hour = ref<number>(0)
const minute = ref<number>(0)
const second = ref<number>(0)

const dateValue = computed(() => {
  let date = props.value ? props.value : props.modelValue
  return date ? dayjsLocal(date).format('YYYY-MM-DD HH:mm:ss') : ''
})

function changeTime() {
  let date = dateValue.value ? getDate(dateValue.value) : getDate()
  selectDate(date)
}

function selectDate(date: Date | string, isNow: boolean = false) {
  if (isNow) {
    date = dayjsLocal(date)
    hour.value = date.hour()
    minute.value = date.minute()
    second.value = 0
  }

  let systemParsedDate = date
    ? dayjsSystem(toValue(date)).format('YYYY-MM-DD HH:mm:ss')
    : ''
  emit('change', systemParsedDate)
  emit('update:modelValue', systemParsedDate)
}

function toValue(date: Date | string) {
  if (!date || date.toString() === 'Invalid Date') return ''

  return dayjs(date)
    .set('hour', hour.value)
    .set('minute', minute.value)
    .set('second', second.value)
    .format('YYYY-MM-DD HH:mm:ss')
}

function twoDigit(number: number) {
  return number.toString().padStart(2, '0')
}

function updateDate(date: Date | string) {
  date = getDate(date)
  hour.value = date.getHours()
  minute.value = date.getMinutes()
  second.value =0
  selectDate(date)
}

function selectCurrentMonthYear() {
  let date = dateValue.value ? getDate(dateValue.value) : getDate()
  if (date.toString() === 'Invalid Date') {
    date = getDate()
  }
  currentYear.value = date.getFullYear()
  currentMonth.value = date.getMonth() + 1
  hour.value = date.getHours()
  minute.value = date.getMinutes()
  second.value = 0
}

function increaseHour() {
  hour.value = (hour.value + 1) % 24
  changeTime()
}

function decreaseHour() {
  hour.value = (hour.value + 23) % 24
  changeTime()
}

function increaseMinute() {
  minute.value = (minute.value + 1) % 60
  changeTime()
}

function decreaseMinute() {
  minute.value = (minute.value + 59) % 60
  changeTime()
}

function increaseSecond() {
  second.value = (second.value + 1) % 60
  changeTime()
}

function decreaseSecond() {
  second.value = (second.value + 59) % 60
  changeTime()
}

onMounted(() => selectCurrentMonthYear())
</script>

<style scoped>
.slider {
  --trackHeight: 1px;
  --thumbRadius: 10px;
}
/* Removed old slider styles */
</style>
