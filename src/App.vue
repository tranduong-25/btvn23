<template>
  <div id="app">
    <div class="employee-container">
      <div class="header">
      </div>
      <EmployeeTable :employees="filteredEmployees" @add-employee="addEmployee" />
      <AddEmployeeForm v-if="showForm" @close="showForm = false" @submit="handleAddEmployee" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'; 
import EmployeeTable from './components/EmployeeTable.vue'; 
const showForm = ref(false);
const searchQuery = ref("");
const employees = ref([ 
  {
    id: 1,
    name: "Nguyễn Văn A",
    dob: "28/02/1990",
    email: "nvana@gmail.com",
    address: "Ba Đình, Hà Nội",
    status: "Đang hoạt động",
  },
  {
    id: 2,
    name: "Trần Thị B",
    dob: "15/07/1985",
    email: "ttb@gmail.com",
    address: "Cầu Giấy, Hà Nội",
    status: "Ngừng hoạt động",
  },
  {
    id: 3,
    name: "Lê Văn C",
    dob: "03/10/2000",
    email: "lvc@gmail.com",
    address: "Hai Bà Trưng, Hà Nội",
    status: "Ngừng hoạt động",
  },
  {
    id: 4,
    name: "Phạm Thị D",
    dob: "20/05/1995",
    email: "ptd@gmail.com",
    address: "Hoàn Kiếm, Hà Nội",
    status: "Đang hoạt động",
  },
  {
    id: 5,
    name: "Ngô Văn E",
    dob: "12/11/1988",
    email: "nve@gmail.com",
    address: "Cầu Giấy, Hà Nội",
    status: "Đang hoạt động",
  },
]);
const filteredEmployees = computed(() => {
  return employees.value.filter(employee =>
    employee.email.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});
const toggleForm = () => {
  showForm.value = !showForm.value;
};
const handleAddEmployee = (newEmployee) => {
  employees.value.push(newEmployee);
  showForm.value = false; 
};

</script>

<style scoped>
#app {
  margin: 0;
  padding: 0;
  width: 100vw; 
  height: 100vh; 
  display: flex;
  justify-content: center;
  align-items: flex-start; 
  background-color: #eaeaea; 
}
.employee-container {
  width: 100%; 
  height: auto; 
  padding: 20px; 
  box-sizing: border-box;
}

h1 {
  font-size: 24px; 
  color: black; 
  margin: 0; 
}

table {
  width: 100%;
  border-collapse: collapse; 
}

table th,
table td {
  text-align: left; 
  padding: 10px; 
  border: 1px solid #ddd; 
  color: black; 
}

table th {
  background-color: #f8f8f8; 
  font-weight: bold; 
}

.status {
  display: inline-block; 
  padding: 5px 10px; 
  border-radius: 20px; 
  color: white; 
  font-size: 12px;
}

.status.active {
  background-color: #28a745; 
}

.status.inactive {
  background-color: #dc3545; 
}


</style>
