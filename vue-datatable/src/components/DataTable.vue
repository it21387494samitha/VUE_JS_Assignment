<template>
  <div class="datatable-container">
    <div class="controls">
      <input v-model="searchQuery" @input="debouncedFilterData" placeholder="Search..." class="search-input">
      <select v-model="rowsPerPage" @change="changeRowsPerPage" class="rows-select">
        <option value="10">10</option>
        <option value="15">15</option>
        <option value="20">20</option>
        <option value="all">All</option>
      </select>
    </div>
    <div v-if="loading" class="loading">Loading...</div>
    <div v-else>
      <table class="datatable">
        <thead>
          <tr>
            <th @click="sortData('id')">ID <span :class="sortKey === 'id' ? (sortAsc ? 'asc' : 'desc') : ''"></span></th>
            <th>Name</th>
            <th @click="sortData('email')">Email <span :class="sortKey === 'email' ? (sortAsc ? 'asc' : 'desc') : ''"></span></th>
            <th>Body</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="comment in paginatedData" :key="comment.id">
            <td>{{ comment.id }}</td>
            <td v-if="!editId || editId !== comment.id">{{ comment.name }}</td>
            <td v-if="editId === comment.id"><input v-model="editComment.name" class="edit-input" /></td>
            <td>{{ comment.email }}</td>
            <td v-if="!editId || editId !== comment.id">{{ comment.body }}</td>
            <td v-if="editId === comment.id"><input v-model="editComment.body" class="edit-input" /></td>
            <td>
              <button @click="startEdit(comment)" v-if="!editId || editId !== comment.id" class="action-button">Edit</button>
              <button @click="saveEdit(comment.id)" v-if="editId === comment.id" class="action-button save-button">Save</button>
              <button @click="deleteComment(comment.id)" class="action-button delete-button">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
      <div class="pagination">
        <button @click="prevPage" :disabled="page === 1" class="pagination-button">Previous</button>
        <button @click="nextPage" :disabled="page === totalPages" class="pagination-button">Next</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const comments = ref([]);
    const searchQuery = ref('');
    const page = ref(1);
    const rowsPerPage = ref(10);
    const sortKey = ref('');
    const sortAsc = ref(true);
    const loading = ref(true);
    const editId = ref(null);
    const editComment = ref({});

    const fetchComments = async () => {
      loading.value = true;
      const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
      comments.value = response.data;
      loading.value = false;
    };

    onMounted(fetchComments);

    const debouncedFilterData = debounce(() => {
      filterData();
    }, 300);

    const filterData = () => {
      page.value = 1;
    };

    const changeRowsPerPage = () => {
      page.value = 1;
    };

    const sortData = (key) => {
      if (sortKey.value === key) {
        sortAsc.value = !sortAsc.value;
      } else {
        sortKey.value = key;
        sortAsc.value = true;
      }
    };

    const filteredData = computed(() => {
      return comments.value.filter(comment => {
        return comment.name.includes(searchQuery.value) || comment.email.includes(searchQuery.value);
      });
    });

    const sortedData = computed(() => {
      return filteredData.value.sort((a, b) => {
        if (!sortKey.value) return 0;
        const compareA = a[sortKey.value];
        const compareB = b[sortKey.value];
        if (compareA < compareB) return sortAsc.value ? -1 : 1;
        if (compareA > compareB) return sortAsc.value ? 1 : -1;
        return 0;
      });
    });

    const paginatedData = computed(() => {
      if (rowsPerPage.value === 'all') return sortedData.value;
      const start = (page.value - 1) * rowsPerPage.value;
      const end = start + rowsPerPage.value;
      return sortedData.value.slice(start, end);
    });

    const totalPages = computed(() => {
      if (rowsPerPage.value === 'all') return 1;
      return Math.ceil(filteredData.value.length / rowsPerPage.value);
    });

    const prevPage = () => {
      if (page.value > 1) page.value--;
    };

    const nextPage = () => {
      if (page.value < totalPages.value) page.value++;
    };

    const startEdit = (comment) => {
      editId.value = comment.id;
      editComment.value = { ...comment };
    };

    const saveEdit = (id) => {
      const index = comments.value.findIndex(c => c.id === id);
      if (index !== -1) {
        comments.value[index] = { ...editComment.value };
      }
      editId.value = null;
    };

    const deleteComment = (id) => {
      comments.value = comments.value.filter(comment => comment.id !== id);
    };

    function debounce(func, wait) {
      let timeout;
      return function(...args) {
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(this, args), wait);
      };
    }

    return {
      searchQuery,
      rowsPerPage,
      page,
      loading,
      paginatedData,
      totalPages,
      sortData,
      prevPage,
      nextPage,
      debouncedFilterData,
      changeRowsPerPage,
      startEdit,
      saveEdit,
      deleteComment,
      editId,
      editComment,
    };
  },
};
</script>

<style scoped>
.datatable-container {
  margin: 20px;
}

.controls {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.search-input, .rows-select {
  padding: 8px;
  font-size: 14px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background-color: var(--background-color);
  color: var(--text-color);
}

.loading {
text-align: center;
font-size: 18px;
padding: 20px;
display: flex;
justify-content: center;
align-items: center;
}

.loading::before {
content: '';
width: 20px;
height: 20px;
border-radius: 50%;
border: 2px solid #ccc;
border-top-color: #007bff;
animation: spin 1s linear infinite;
margin-right: 10px;
}

@keyframes spin {
0% {
  transform: rotate(0deg);
}
100% {
  transform: rotate(360deg);
}
}


.datatable {
  width: 100%;
  border-collapse: collapse;
}

.datatable th, .datatable td {
  padding: 10px;
  border: 1px solid var(--border-color);
  text-align: left;
}

.datatable th {
  cursor: pointer;
  background-color: var(--header-background-color);
}

.datatable th.asc::after {
  content: ' ▲';
}

.datatable th.desc::after {
  content: ' ▼';
}

.edit-input {
  padding: 5px;
  font-size: 14px;
  background-color: var(--background-color);
  color: var(--text-color);
}

.action-button {
  padding: 5px 10px;
  margin: 0 5px;
  background-color: var(--action-button-background-color);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 5px;
  width: 70px;
}

.action-button:hover {
  background-color: var(--action-button-hover-background-color);
}

.delete-button {
  background-color: var(--delete-button-background-color);
}
.save-button{
  background-color: var(--save-button-background-color);
}
.save-button:hover{
  background-color: var(--save-button-hover-background-color);
}

.pagination {
  margin-top: 10px;
  display: flex;
  justify-content: center;
}

.pagination-button {
  padding: 8px 16px;
  margin: 0 5px;
  border: 1px solid var(--border-color);
  background-color: var(--header-background-color);
  cursor: pointer;
}

.pagination-button:disabled {
  background-color: #e9ecef;
  cursor: not-allowed;
}
</style>
