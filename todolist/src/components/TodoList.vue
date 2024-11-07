<template>
  <div class="p-6 bg-gray-100 min-h-screen">
    <h1 class="text-2xl font-bold mb-4">To-Do List</h1>
    <button 
      @click="showAddModal = true" 
      class="bg-blue-500 text-white px-4 py-2 rounded mb-4 hover:bg-blue-600">
      Thêm công việc
    </button>

    <div v-if="showAddModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
  <div class="bg-white p-6 rounded shadow-lg w-96">
    <h3 class="text-xl font-semibold mb-4">Thêm công việc mới</h3>
    <input
      v-model="newTodo.text"
      type="text"
      placeholder="Nhập công việc..."
      class="p-2 border rounded w-full mb-2"
    />
    <input
      v-model="newTodo.date"
      type="datetime-local"
      class="p-2 border rounded w-full mb-2"
    />
    <div class="flex justify-end mt-4">
      <button @click="addTodo" class="bg-blue-500 text-white px-4 py-2 rounded mr-2 hover:bg-blue-600">
        Thêm
      </button>
      <button @click="closeModal" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600">
        Hủy
      </button>
    </div>
  </div>
</div>

    <div class="mb-6 flex space-x-4">
      <input
        v-model="searchTerm"
        type="text"
        placeholder="Tìm kiếm công việc..."
        class="p-2 border rounded w-full"
      />
      <select v-model="statusFilter" class="p-2 border rounded">
        <option value="">Tất cả trạng thái</option>
        <option value="Đang diễn ra">Đang diễn ra</option>
        <option value="Đã hoàn thành">Đã hoàn thành</option>
        <option value="Đã quá hạn">Đã quá hạn</option>
      </select>
    </div>

    <TodoTable 
      :todos="filteredTodos" 
      @edit-todo="openEditModal" 
      @delete-todo="openDeleteModal" 
      @complete-selected="completeSelectedTodos" 
    />

    <button 
      v-if="hasSelectedTodos" 
      @click="confirmCompletionModal" 
      class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600">
      Đã hoàn thành
    </button>

    <!-- <Calendar :todos="todos" /> calendar bi loi -->

    <div v-if="showCompletionModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white p-6 rounded shadow-lg w-96">
        <h3 class="text-xl font-semibold mb-4">Xác nhận hoàn thành công việc</h3>
        <p>Bạn có chắc chắn muốn đánh dấu công việc này là hoàn thành?</p>
        <div class="flex justify-end mt-4">
          <button @click="completeSelectedTodos" class="bg-green-500 text-white px-4 py-2 rounded mr-2 hover:bg-green-600">Hoàn thành</button>
          <button @click="closeModal" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600">Hủy</button>
        </div>
      </div>
    </div>


    <div v-if="showEditModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white p-6 rounded shadow-lg w-96">
        <h3 class="text-xl font-semibold mb-4">Chỉnh sửa công việc</h3>
        <input
          v-model="editTodo.text"
          type="text"
          class="p-2 border rounded w-full mb-2"
          placeholder="Sửa công việc..."
        />
        <input v-model="editTodo.date" type="datetime-local" class="p-2 border rounded w-full mb-2" />
        <div class="flex justify-end">
          <button @click="updateTodo" class="bg-green-500 text-white px-4 py-2 rounded mr-2 hover:bg-green-600">
            Lưu
          </button>
          <button @click="closeModal" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600">
            Hủy
          </button>
        </div>
      </div>
    </div>

    <div v-if="showDeleteModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white p-6 rounded shadow-lg w-96">
        <h3 class="text-xl font-semibold mb-4">Xác nhận xóa công việc</h3>
        <p>Bạn có chắc chắn muốn xóa công việc này?</p>
        <div class="flex justify-end mt-4">
          <button @click="deleteTodo" class="bg-red-500 text-white px-4 py-2 rounded mr-2 hover:bg-red-600">
            Xóa
          </button>
          <button @click="closeModal" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600">
            Hủy
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import TodoTable from './TodoTable.vue'
import Calendar from './Calendar.vue'

export default {
  components: {
    TodoTable,
    Calendar,
  },
  setup() {
    const newTodo = ref({ text: '', date: '', status: 'Đang diễn ra' });
    const todos = ref([]);
    const searchTerm = ref('');
    const statusFilter = ref('');
    const showAddModal = ref(false);

    const showEditModal = ref(false);
    const showDeleteModal = ref(false);
    const showCompletionModal = ref(false);
    const editTodoIndex = ref(null);
    const editTodo = ref({ text: '', date: '', status: 'Đang diễn ra' });

    const hasSelectedTodos = computed(() => todos.value.some(todo => todo.selected));

    const confirmCompletionModal = () => {
      if (todos.value.some(todo => todo.selected)) {
        showCompletionModal.value = true;
      } else {
        alert("Chọn ít nhất một công việc để hoàn thành.");
      }
    };

    const addTodo = () => {
  if (newTodo.value.text && newTodo.value.date) {
    const now = new Date();
    const todoDate = new Date(newTodo.value.date);
    const todoStatus = todoDate < now ? 'Đã quá hạn' : 'Đang diễn ra';

    todos.value.push({ 
      ...newTodo.value, 
      status: todoStatus, 
      id: Date.now() 
    });
    newTodo.value = { text: '', date: '', status: 'Đang diễn ra' };
    showAddModal.value = false; 
    saveTodosToLocalStorage(); 
    checkOverdueTasks(); 
  } else {
    alert('Vui lòng nhập đủ thông tin!');
  }
};



    const openEditModal = (todoId) => {
      const todo = todos.value.find((item) => item.id === todoId);
      if (todo) {
        editTodoIndex.value = todoId; 
        editTodo.value = { ...todo };
        showEditModal.value = true;
      }
    };

    
    const checkOverdueTasks = () => {
      const now = new Date();
      todos.value.forEach((todo) => {
        const todoDate = new Date(todo.date); 
        if (todo.status === 'Đang diễn ra' && todoDate < now) {
          todo.status = 'Đã quá hạn'; 
        } else if (todo.status === 'Đã quá hạn' && todoDate > now) {
          todo.status = 'Đang diễn ra'; 
        }
      });
    };


    const loadTodosFromLocalStorage = () => {
      const storedTodos = JSON.parse(localStorage.getItem('todos'));
      if (storedTodos) {
        todos.value = storedTodos;
      }
    };

    const saveTodosToLocalStorage = () => {
      localStorage.setItem('todos', JSON.stringify(todos.value));
    };

    let intervalId;
    onMounted(() => {
      loadTodosFromLocalStorage();
      checkOverdueTasks();
      intervalId = setInterval(checkOverdueTasks, 60000); 
    });

    onUnmounted(() => {
      clearInterval(intervalId);
    });

    const updateTodo = () => {
  if (editTodoIndex.value !== null) {
    const newDate = new Date(editTodo.value.date);
    const now = new Date();

    const todoIndex = todos.value.findIndex(todo => todo.id === editTodoIndex.value);
    if (todoIndex !== -1) {
      todos.value[todoIndex] = { ...editTodo.value };
      todos.value[todoIndex].status = newDate > now ? 'Đang diễn ra' : 'Đã quá hạn';
      
      saveTodosToLocalStorage();
      checkOverdueTasks();
      closeModal(); 
    }
  }
};

    const deleteTodo = () => {
      if (editTodoIndex.value !== null) {
        const todoIndex = todos.value.findIndex(todo => todo.id === editTodoIndex.value);
        if (todoIndex !== -1) {
          todos.value.splice(todoIndex, 1);
          saveTodosToLocalStorage();
          closeModal();
        }
      }
    };

    const openDeleteModal = (todoId) => {
      editTodoIndex.value = todoId; 
      showDeleteModal.value = true;
    }

    const completeSelectedTodos = () => {
      todos.value.forEach(todo => {
        if (todo.selected) {
          todo.status = 'Đã hoàn thành';
          todo.selected = false;
        }
      });
      showCompletionModal.value = false;
      saveTodosToLocalStorage();
    };

    const confirmCompletion = (index) => {
      editTodoIndex.value = index;
      showCompletionModal.value = true;
    }

    const completeTodo = () => {
      if (editTodoIndex.value !== null) {
        todos.value[editTodoIndex.value].status = 'Đã hoàn thành';
        closeModal();
      }
    }

    const closeModal = () => {
      showAddModal.value = false;
      showEditModal.value = false;
      showDeleteModal.value = false;
      showCompletionModal.value = false;
      editTodoIndex.value = null;
      editTodo.value = { text: '', date: '', status: 'Đang diễn ra' };
    }

    const filteredTodos = computed(() => {
      return todos.value.filter(todo => {
        const matchesSearch = todo.text.toLowerCase().includes(searchTerm.value.toLowerCase());
        const matchesStatus = statusFilter.value === '' || todo.status === statusFilter.value;
        return matchesSearch && matchesStatus;
      });
    });

    return {
      filteredTodos,
      hasSelectedTodos,
      newTodo,
      todos,
      showAddModal,
      showEditModal,
      showDeleteModal,
      showCompletionModal,
      statusFilter,
      searchTerm,
      editTodo,
      addTodo,
      confirmCompletionModal,
      openEditModal,
      updateTodo,
      openDeleteModal,
      deleteTodo,
      confirmCompletion,
      completeTodo,
      completeSelectedTodos,
      closeModal,
    }
  },
}
</script>

<style scoped>

</style>
