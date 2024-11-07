<script>
export default {
  props: {
      todos: Array,
  },
  data() {
      return {
          allSelected: false,
          selectedCount: 0,
          currentPage: 1,
          itemsPerPage: 6,
      };
  },
  computed: {
    paginatedTodos() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.todos.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.todos.length / this.itemsPerPage);
    },
  },
  methods: {
    toggleSelectAll() {
      this.todos.forEach(todo => {
        if (todo.status !== 'Đã hoàn thành') {
          todo.selected = this.allSelected;
        }
      });
    },
      updateSelectedCount() {
          this.selectedCount = this.todos.filter(todo => todo.selected).length;
          this.allSelected = this.selectedCount === this.todos.length;
      },
      nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        this.allSelected = false;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        this.allSelected = false;
      }
    },
    goToPage(page) {
      this.currentPage = page;
      this.allSelected = false;
    },
  },
}
</script>

<template>
  <div class="mt-8">
      <h2 class="text-xl font-semibold mb-4">Danh sách công việc</h2>
      <table class="min-w-full bg-white border rounded">
          <thead>
              <tr class="bg-gray-200 text-gray-600 uppercase text-sm leading-normal">
                  <th class="py-3 px-6 text-left">
                    <input
                        type="checkbox"
                        v-model="allSelected"
                        @change="toggleSelectAll"
                        class="appearance-none h-5 w-5 border border-gray-800 rounded-full checked:bg-blue-500 checked:border-transparent transition-all duration-200 cursor-pointer"
                    />
                  </th>
                  <th class="py-3 px-6 text-left">STT</th>
                  <th class="py-3 px-6 text-left">Công việc</th>
                  <th class="py-3 px-6 text-left">Ngày</th>
                  <th class="py-3 px-6 text-left">Trạng thái</th>
                  <th class="py-3 px-6 text-center">Hành động</th>
              </tr>
          </thead>
          <tbody class="text-gray-700 text-sm">
              <tr v-for="(todo, index) in paginatedTodos" :key="todo.id" class="border-b border-gray-200 hover:bg-gray-100">
                  <td class="py-3 px-6 text-left">
                      <input type="checkbox" v-model="todo.selected" @change="updateSelectedCount" :disabled="todo.status === 'Đã hoàn thành'"  class="appearance-none h-5 w-5 border border-gray-800 rounded-full checked:bg-blue-500 checked:border-transparent transition-all duration-200 cursor-pointer
                      disabled:opacity-50 disabled:cursor-not-allowed" />
                  </td>
                  <td class="py-3 px-6 text-left">{{ index + 1 + (currentPage - 1) * itemsPerPage }}</td>
                  <td class="py-3 px-6 text-left">{{ todo.text }}</td>
                  <td class="py-3 px-6 text-left">{{ todo.date }}</td>
                  <td class="py-3 px-6 text-left">{{ todo.status }}</td>
                  <td class="py-3 px-6 text-center">
                      <button
                          v-if="todo.status !== 'Đã hoàn thành'"
                          @click="$emit('edit-todo', todo.id)"
                          class="bg-yellow-500 text-white px-3 py-1 rounded mr-2 hover:bg-yellow-600"
                      >
                          Sửa
                      </button>
                      <button
                          @click="$emit('delete-todo', todo.id)"
                          class="bg-red-500 text-white px-3 py-1 rounded mr-2 hover:bg-red-600"
                      >
                          Xóa
                      </button>
                  </td>
              </tr>
          </tbody>
      </table>

      <div class="mt-4 flex justify-center space-x-2">
          <button @click="prevPage" :disabled="currentPage === 1" class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400">
            Trước
          </button>
          <span v-for="page in totalPages" :key="page" @click="goToPage(page)" :class="{'font-bold': page === currentPage}" class="px-2 cursor-pointer">
            {{ page }}
          </span>
          <button @click="nextPage" :disabled="currentPage === totalPages" class="px-4 py-2 bg-gray-300 rounded hover:bg-gray-400">
            Sau
          </button>
      </div>

  </div>
</template>

<style scoped>
</style>