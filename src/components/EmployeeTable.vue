<template>
    <div>
      <h1>Quản lý nhân viên</h1>
      <div class="header">
        <input
          v-model="searchQuery"
          type="text"
          class="search-input"
          placeholder="Tìm kiếm theo email"
        />
        <button class="add-employee-btn" @click="toggleForm">
          {{ showForm ? "Hủy" : "Thêm mới nhân viên" }}
        </button>
      </div>
      <div v-if="showForm" class="form-container">
        <div class="form-header">
          <h2>{{ isEditing ? "Sửa thông tin nhân viên" : "Thêm mới nhân viên" }}</h2>
          <button class="close-button" @click="toggleForm">✖</button>
        </div>
        <form @submit.prevent="submitForm">
          <div>
            <label for="name">Họ và tên:</label>
            <input type="text" id="name" v-model="newEmployee.name" required />
          </div>
          <div>
            <label for="email">Email:</label>
            <input type="email" id="email" v-model="newEmployee.email" required />
          </div>
          <div>
            <label for="dob">Ngày sinh:</label>
            <input type="date" id="dob" v-model="newEmployee.dob" required />
          </div>
          <div>
            <label for="address">Địa chỉ:</label>
            <input type="text" id="address" v-model="newEmployee.address" />
          </div>
          <div>
            <label for="status">Trạng thái:</label>
            <select id="status" v-model="newEmployee.status" required>
              <option value="Đang hoạt động">Hoạt động</option>
              <option value="Ngừng hoạt động">Không hoạt động</option>
            </select>
          </div>
          <span v-if="errorMessage" class="error-message">{{ errorMessage }}</span>
          <button type="submit" class="submit-button">{{ isEditing ? "Cập nhật nhân viên" : "Thêm nhân viên" }}</button>
        </form>
      </div>
  
      <table class="employee-table">
        <thead>
          <tr>
            <th>STT</th>
            <th>Họ và tên</th>
            <th>Ngày sinh</th>
            <th>Email</th>
            <th>Địa chỉ</th>
            <th>Trạng thái</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(employee, index) in filteredEmployees" :key="employee.id">
            <td>{{ index + 1 }}</td>
            <td>{{ employee.name }}</td>
            <td>{{ employee.dob }}</td>
            <td>{{ employee.email }}</td>
            <td>{{ employee.address }}</td>
            <td>
              <span :class="statusClass(employee.status)">{{ employee.status }}</span>
            </td>
            <td>
              <button v-if="employee.status === 'Ngừng hoạt động'" class="unblock-btn" @click="showConfirmModal('unblock', employee.id)">
                Bỏ chặn
              </button>
              <button v-else class="block-btn" @click="showConfirmModal('block', employee.id)">
                Chặn
              </button>
              <button class="edit-btn" @click="editEmployee(employee)">Sửa</button>
              <button class="delete-btn" @click="showConfirmModal('delete', employee.id)">Xóa</button>
            </td>
          </tr>
        </tbody>
      </table>
  
      <div v-if="showModal" class="modal">
        <div class="modal-content">
          <span class="close" @click="closeModal">✖</span>
          <h3>{{ modalTitle }}</h3>
          <p>Bạn có chắc chắn muốn {{ modalAction }} nhân viên này?</p>
          <button @click="confirmAction">Xác nhận</button>
          <button @click="closeModal">Hủy</button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed } from "vue";
  
  const employees = ref([
    {
      id: 1,
      name: "Nguyễn Văn A",
      dob: "1990-02-28",
      email: "nvana@gmail.com",
      address: "Ba Đình, Hà Nội",
      status: "Đang hoạt động",
    },
    {
      id: 2,
      name: "Trần Thị B",
      dob: "1985-07-15",
      email: "ttb@gmail.com",
      address: "Cầu Giấy, Hà Nội",
      status: "Ngừng hoạt động",
    },
    {
      id: 3,
      name: "Lê Văn C",
      dob: "2000-10-03",
      email: "lvc@gmail.com",
      address: "Hai Bà Trưng, Hà Nội",
      status: "Ngừng hoạt động",
    },
    {
      id: 4,
      name: "Phạm Thị D",
      dob: "1995-05-20",
      email: "ptd@gmail.com",
      address: "Hoàn Kiếm, Hà Nội",
      status: "Đang hoạt động",
    },
    {
      id: 5,
      name: "Ngô Văn E",
      dob: "1988-11-12",
      email: "nve@gmail.com",
      address: "Cầu Giấy, Hà Nội",
      status: "Đang hoạt động",
    },
  ]);
  
  const searchQuery = ref("");
  const showForm = ref(false);
  const newEmployee = ref({
    name: "",
    email: "",
    dob: "",
    address: "",
    status: "Đang hoạt động",
  });
  const errorMessage = ref("");
  const showModal = ref(false);
  const modalAction = ref("");
  const modalTitle = ref("");
  const selectedEmployeeId = ref(null);
  const isEditing = ref(false);
  
  const filteredEmployees = computed(() => {
    return employees.value.filter((employee) =>
      employee.email.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  });
  
  const toggleForm = () => {
    showForm.value = !showForm.value;
    if (!showForm.value) resetForm();
  };
  
  const isValidEmail = (email) => {
    const emailPattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    return emailPattern.test(email);
  };
  
  const validateEmployeeData = () => {
    if (!newEmployee.value.name || !newEmployee.value.email) {
      return "Họ và tên và Email không được phép để trống.";
    }
    if (!isValidEmail(newEmployee.value.email)) {
      return "Email không hợp lệ.";
    }
    if (employees.value.some(emp => emp.email === newEmployee.value.email && emp.id !== (isEditing.value ? newEmployee.value.id : undefined))) {
      return "Email đã tồn tại.";
    }
    const today = new Date();
    const dob = new Date(newEmployee.value.dob);
    if (dob > today) {
      return "Ngày sinh không được lớn hơn ngày hiện tại.";
    }
    return null;
  };
  
  const submitForm = async () => {
    const validationError = validateEmployeeData();
    if (validationError) {
      errorMessage.value = validationError;
      return;
    }
    errorMessage.value = "";
  
    if (isEditing.value) {
      const index = employees.value.findIndex(emp => emp.id === newEmployee.value.id);
      if (index !== -1) {
        employees.value[index] = { ...newEmployee.value }; 
      }
    } else {
      const employeeToAdd = {
        ...newEmployee.value,
        id: Date.now(),
      };
      employees.value.push(employeeToAdd);
    }
  
    resetForm();
    showForm.value = false;
  };
  
  const resetForm = () => {
    newEmployee.value = {
      name: "",
      email: "",
      dob: "",
      address: "",
      status: "Đang hoạt động",
    };
    isEditing.value = false; 
  };
  
  const editEmployee = (employee) => {
    newEmployee.value = { ...employee };
    isEditing.value = true; 
    showForm.value = true; 
  };
  
  const deleteEmployee = (id) => {
    employees.value = employees.value.filter(emp => emp.id !== id);
  };
  
  const showConfirmModal = (action, id) => {
    modalAction.value = action === 'block' ? 'chặn' : (action === 'unblock' ? 'bỏ chặn' : 'xóa');
    modalTitle.value = `Cảnh báo`;
    selectedEmployeeId.value = id;
    showModal.value = true;
  };
  
  const confirmAction = () => {
    if (modalAction.value === 'chặn') {
      blockEmployee(selectedEmployeeId.value);
    } else if (modalAction.value === 'unblock') {
      unblockEmployee(selectedEmployeeId.value);
    } else if (modalAction.value === 'xóa') {
      deleteEmployee(selectedEmployeeId.value);
    }
    closeModal();
  };
  
  const closeModal = () => {
    showModal.value = false;
    selectedEmployeeId.value = null;
    modalAction.value = "";
    modalTitle.value = "";
  };
  
  const statusClass = (status) => {
    return status === "Đang hoạt động" ? "status-active" : "status-inactive";
  };
  
  const blockEmployee = (id) => {
    const index = employees.value.findIndex(emp => emp.id === id);
    if (index !== -1) {
      employees.value[index].status = "Ngừng hoạt động";
    }
  };
  
  const unblockEmployee = (id) => {
    const index = employees.value.findIndex(emp => emp.id === id);
    if (index !== -1) {
      employees.value[index].status = "Đang hoạt động";
    }
  };
  </script>
  
  <style scoped>
  .header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
  }
  
  .search-input {
    padding: 10px;
    width: 300px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  .add-employee-btn,
  .submit-button,
  .edit-btn,
  .delete-btn,
  .block-btn,
  .unblock-btn {
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .add-employee-btn {
    background-color: #28a745;
    color: white;
  }
  
  .submit-button {
    background-color: #007bff;
    color: white;
  }
  
  .edit-btn {
    background-color: #ffc107;
    color: white;
  }
  
  .delete-btn {
    background-color: #dc3545;
    color: white;
  }
  
  .block-btn {
    background-color: #ffc107;
    color: white;
  }
  
  .unblock-btn {
    background-color: #28a745;
    color: white;
  }
  
  .form-container {
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 20px;
    margin-bottom: 20px;
  }
  
  .form-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .close-button {
    background: none;
    border: none;
    font-size: 20px;
    cursor: pointer;
  }
  
  .employee-table {
    width: 100%;
    border-collapse: collapse;
  }
  
  .employee-table th,
  .employee-table td {
    border: 1px solid #ccc;
    padding: 10px;
    text-align: left;
  }
  
  .error-message {
    color: red;
  }
  
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .modal-content {
    background-color: white;
    padding: 20px;
    border-radius: 4px;
    text-align: center;
  }
  
  .close {
    cursor: pointer;
    font-size: 24px;
    position: absolute;
    top: 10px;
    right: 10px;
  }
  
  .status-active {
    color: green;
  }
  
  .status-inactive {
    color: red;
  }
  </style>
  