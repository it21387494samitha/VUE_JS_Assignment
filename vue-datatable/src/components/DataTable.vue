<template>
    <div>
      <input v-model="searchQuery" @input="filterData" placeholder="Search...">
      <table>
        <thead>
          <tr>
            <th @click="sortData('id')">ID</th>
            <th>Name</th>
            <th @click="sortData('email')">Email</th>
            <th>Body</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="comment in paginatedData" :key="comment.id">
            <td>{{ comment.id }}</td>
            <td>{{ comment.name }}</td>
            <td>{{ comment.email }}</td>
            <td>{{ comment.body }}</td>
          </tr>
        </tbody>
      </table>
      <button @click="prevPage" :disabled="page === 1">Previous</button>
      <button @click="nextPage" :disabled="page === totalPages">Next</button>
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
      const perPage = 10;
      const sortKey = ref('');
      const sortAsc = ref(true);
  
      const fetchComments = async () => {
        const response = await axios.get('https://jsonplaceholder.typicode.com/comments');
        comments.value = response.data;
      };
  
      onMounted(fetchComments);
  
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
        const start = (page.value - 1) * perPage;
        const end = start + perPage;
        return sortedData.value.slice(start, end);
      });
  
      const totalPages = computed(() => {
        return Math.ceil(filteredData.value.length / perPage);
      });
  
      const filterData = () => {
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
  
      const prevPage = () => {
        if (page.value > 1) page.value--;
      };
  
      const nextPage = () => {
        if (page.value < totalPages.value) page.value++;
      };
  
      return {
        comments,
        searchQuery,
        page,
        paginatedData,
        totalPages,
        filterData,
        sortData,
        prevPage,
        nextPage,
      };
    },
  };
  </script>
  
  <style scoped>
  table {
    width: 100%;
    border-collapse: collapse;
  }
  
  th, td {
    padding: 8px;
    border: 1px solid #ddd;
  }
  
  th {
    cursor: pointer;
  }
  
  input {
    margin-bottom: 10px;
    padding: 5px;
  }
  
  button {
    margin: 5px;
  }
  </style>
  