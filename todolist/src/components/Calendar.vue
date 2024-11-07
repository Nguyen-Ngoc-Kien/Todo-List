<!-- Calendar dang bi loi~ -->
<template>
  <div class="mt-8">
    <h2 class="text-xl font-semibold mb-4">Lịch công việc</h2>

    <div class="flex space-x-4 mb-4">
      <select v-model="selectedMonth" class="p-2 border rounded">
        <option v-for="(month, index) in months" :key="index" :value="index + 1">
          {{ month }}
        </option>
      </select>
      <select v-model="selectedYear" class="p-2 border rounded">
        <option v-for="year in years" :key="year" :value="year">
          {{ year }}
        </option>
      </select>
    </div>

    <div class="grid grid-cols-7 gap-4">
      <div
        v-for="day in daysInSelectedMonth"
        :key="day"
        class="border p-2 rounded bg-gray-50 min-h-24 relative"
      >
        <div class="text-xs font-bold mb-2">{{ day }}</div>
        <ul class="text-xs">
          <li
            v-for="todo in todosByDate(day)"
            :key="todo.text"
            :class="{
              'text-blue-500': todo.status === 'Đã hoàn thành',
              'text-green-500': todo.status === 'Đang diễn ra',
              'text-red-500': todo.status === 'Đã quá hạn'
            }"
          >
            {{ todo.text }} - <span class="font-semibold">{{ todo.status }}</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { computed, ref } from 'vue'

export default {
  props: {
    todos: {
      type: Array,
      required: true,
    },
  },
  setup(props) {
    const months = [
      'Tháng 1', 'Tháng 2', 'Tháng 3', 'Tháng 4', 'Tháng 5', 'Tháng 6',
      'Tháng 7', 'Tháng 8', 'Tháng 9', 'Tháng 10', 'Tháng 11', 'Tháng 12'
    ]
    const years = Array.from({ length: 5 }, (_, i) => new Date().getFullYear() - 2 + i)

    const selectedMonth = ref(new Date().getMonth() + 1)
    const selectedYear = ref(new Date().getFullYear())

    const daysInSelectedMonth = computed(() => {
      return Array.from({ length: new Date(selectedYear.value, selectedMonth.value, 0).getDate() }, (_, i) => i + 1);
    })

    const todosByDate = (day) => {
      const dayString = `${selectedYear.value}-${String(selectedMonth.value).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
      return props.todos.filter((todo) => todo.date === dayString);
    }

    return {
      months,
      years,
      selectedMonth,
      selectedYear,
      daysInSelectedMonth,
      todosByDate,
    }
  },
}
</script>

<style scoped>

</style>