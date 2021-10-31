<script setup lang="ts">
import { ref, watch, computed } from 'vue'

const INITIAL_TIME = '00:00:00'
const time = ref(INITIAL_TIME)
const sec = ref(0)
const startSec = ref(0)
const timerId = ref(0)
const isTimerStopped = ref(true)

const HOUR_IN_SEC = 3600
const MIN_IN_SEC = 60
const timeToSec = (t: string): number => {
  const [h,m,s] = t.split(':').map(e => parseInt(e))
  if (s === undefined) return 0
  return h * HOUR_IN_SEC + m * MIN_IN_SEC + s
}
const secToTime = (s: number): string => {
  return new Date(s * 1000).toISOString().substr(11, 8)
}
const countDownInner = (): void => {
  sec.value--
  time.value = secToTime(sec.value)
}
const countDown = (): void => {
  timerId.value = setInterval(countDownInner, 1000)
}
const resumeTimer = (): void => {
  if (sec.value <= 0) return
  isTimerStopped.value = false
  countDown()
}
const startTimer = (): void => {
  sec.value = timeToSec(time.value)
  startSec.value = sec.value
  resumeTimer()
}
const stopTimer = (): void => {
  clearInterval(timerId.value)
  isTimerStopped.value = true
}
const resetTimer = (): void => {
  stopTimer()
  time.value = INITIAL_TIME
  startSec.value = 0
}
watch(sec, (): void => {
  if (!isTimerStopped.value && sec.value > 0) return
  resetTimer()
})
const remainingTimePercent = computed((): number => {
  const st = startSec.value
  if (st === 0) return 0
  return Math.round((st - sec.value) / st * 100)
})
</script>

<template>
  <div class="flex items-center h-screen dark:bg-gray-800">
    <div
      class="shadow-md rounded-md mx-auto dark:bg-gray-800"
      style="width: 350px;">
      <div class="p-5 text-center">
        <h5 class="text-xl font-semibold mb-2 dark:text-gray-200">
          Timer
        </h5>
        <div class="mb-4">
          <input
            v-model="time"
            type="time"
            step="1"
            class="outline-none dark:bg-gray-800 dark:text-gray-200">
        </div>
        <div class="relative mb-4">
          <div class="flex mb-2 items-center justify-between">
            <div>
              <span
                v-if="!isTimerStopped"
                class="
          text-xs
          font-semibold
          inline-block
          px-2
          uppercase
          rounded-full
          text-purple-600
          bg-purple-200
        ">
                Task in progress
              </span>
            </div>
            <div class="text-right">
              <span class="text-xs font-semibold inline-block text-purple-600 py-1">
                {{ remainingTimePercent }}%
              </span>
            </div>
          </div>
          <div class="overflow-hidden h-2 mb-4 text-xs flex rounded bg-purple-200">
            <div
              :style="'width:' + remainingTimePercent + '%'"
              class="
        shadow-none
        flex flex-col
        text-center
        whitespace-nowrap
        text-white
        justify-center
        bg-purple-500
      " />
          </div>
        </div>
        <button
          v-if="!isTimerStopped"
          class="purple-btn"
          type="button"
          @click="stopTimer()">
          Stop
        </button>
        <button
          v-else-if="startSec !== 0"
          class="purple-btn"
          @click="resumeTimer">
          Resume
        </button>
        <button
          v-else
          class="purple-btn"
          type="button"
          @click="startTimer()">
          Start
        </button>
        <button
          class="purple-btn"
          type="button"
          @click="resetTimer()">
          Reset
        </button>
      </div>
    </div>
  </div>
</template>
