<script setup>
import {reactive,ref,toRaw,computed} from'vue'
const activateForm=ref(false);
const checkStatusForm=ref('add');
const users=reactive(JSON.parse(localStorage.getItem('users'))||[])
const initialUser=reactive({id:0,name:'',birthday:'',email:'',address:'',status:true})
const initialError=reactive({name:'',birthday:'',email:''})
const deleteModal=ref(false);
const deleteUser=ref();
const searchQuery=ref('');
const resetError=()=>{
  initialError.name='';
  initialError.birthday='';
  initialError.email=''
}
const resetUser=()=>{
  initialUser.id=0;
  initialUser.name='';
  initialUser.birthday='';
  initialUser.email='';
  initialUser.address='';
  initialUser.status='';
}
// check email có hợp lệ hay ko?
function validateEmail(email) {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
}
// check ngày có vượt qua ngày hiện tại ko???
function validateDate(selectedDate) {
  const currentDate = new Date(); 
  const chosenDate = new Date(selectedDate);
  if (chosenDate > currentDate) {
    return false;
  }
  return true; 
}
const validateUser=()=>{
  if(initialUser.name==''){
    initialError.name='tên không đc để trống';
  }
  if(initialUser.email==''){
    initialError.email='email không đc để trống';
  }else if(!validateEmail(initialUser.email)){
    initialError.email='email không hợp lệ'
  }
  if(initialUser.birthday==''){
    initialError.birthday='ngày sinh không đc để trống'
  }else if(!validateDate(initialUser.birthday)){
    initialError.birthday='ngày sinh không đc vượt qua ngày hiện tại'
  }
}
const handleSubmit=()=>{
  resetError();
  validateUser();
  if(!initialError.name &&!initialError.birthday&&!initialError.email){
    if(checkStatusForm.value=='add'){
      initialUser.id=Math.ceil(Math.random()*100001);
      users.push({...initialUser});
    }else if(checkStatusForm.value=='edit'){
      let findNumber=users.findIndex((item)=>{
        return item.id===initialUser.id;
      })
      users.splice(findNumber,1,{...initialUser});
      checkStatusForm.value=='add';
    }
    activateForm.value=false;
    resetUser();
     localStorage.setItem("users",JSON.stringify(users));
  }
}
const blockUser=(item)=>{
  item.status=!item.status;
  localStorage.setItem("users",JSON.stringify(users));
}
const handleUpdateUser=(item)=>{
  const copiedItem = {...item}
  initialUser.id=copiedItem.id;
  initialUser.name=copiedItem.name;
  initialUser.birthday=copiedItem.birthday;
  initialUser.address=copiedItem.address;
  initialUser.email=copiedItem.email;
  activateForm.value=true;
  checkStatusForm.value='edit';
}
const handleDeleteModal=(item)=>{
  deleteModal.value=true;
  deleteUser.value=item;
}
const handleDelete=()=>{
      let findNumber=users.findIndex((item)=>{
        return item.id===deleteUser.id;
      })
      users.splice(findNumber,1);
       localStorage.setItem("users",JSON.stringify(users));
       deleteModal.value=false;
}
const handleCloseDelete=()=>{
  deleteModal.value=false;
}
const filteredUsers = computed(() => {
  if (searchQuery.value) {
    return users.filter((item)=>{ 
      return item.email.toLowerCase().includes(searchQuery.value.toLowerCase());
    })
  }
  return users;
});
const addUser=()=>{
  activateForm.value=true;
}
const handleCloseForm=()=>{
  activateForm.value=false;
}
</script>
<template>
 <body>
    <div class="w-[80%] m-auto mt-4 h-[100vh]">
      <main class="main">
        <header class="d-flex justify-content-between mb-3">
          <h3>Nhân viên</h3>
          <button class="btn btn-primary" @click='addUser'>Thêm mới nhân viên</button>
        </header>
        <div class="d-flex align-items-center justify-content-end gap-2 mb-3">
          <input
            style="width: 350px"
            type="text"
            class="form-control"
            placeholder="Tìm kiếm theo email"
            v-model="searchQuery"
          />
          <i  class="fa-solid fa-arrows-rotate" title="Refresh"></i>
        </div>
        <!-- Danh sách nhân viên -->
        <table class="table table-bordered table-hover table-striped">
          <thead>
            <tr>
              <th>STT</th>
              <th>Họ và tên</th>
              <th>Ngày sinh</th>
              <th>Email</th>
              <th>Địa chỉ</th>
              <th>Trạng thái</th>
              <th colspan="2">Chức năng</th>
            </tr>
          </thead>
          <tbody>
            <tr :key='item.id' v-for="item in filteredUsers">
              <td>{{item.id}}</td>
              <td>{{item.name}}</td>
              <td>{{item.birthday}}</td>
              <td>{{item.email}}</td>
              <td>{{item.address}}</td>
              <td>
                <div style="display: flex; align-items: center; gap: 8px">
                  <div class="status status-active"></div>
                  <span>{{item.status?'đang hoạt động':'không hoạt động'}}</span>
                </div>
              </td>
              <td>
                <span class="button button-block" @click="blockUser(item)">{{item.status?'Chặn':'Bỏ chặn'}}</span>
              </td>
              <td>
                <span class="button button-edit" @click="handleUpdateUser(item)">Sửa</span>
              </td>
              <td><span class="button button-delete" @click="handleDeleteModal(item)">Xóa</span></td>
            </tr>
          </tbody>
        </table>
        <footer class="d-flex justify-content-end align-items-center gap-3">
          <select class="form-select">
            <option selected>Hiển thị 10 bản ghi trên trang</option>
            <option>Hiển thị 20 bản ghi trên trang</option>
            <option>Hiển thị 50 bản ghi trên trang</option>
            <option>Hiển thị 100 bản ghi trên trang</option>
          </select>
          <ul class="pagination">
            <li class="page-item">
              <a class="page-link" href="#">Previous</a>
            </li>
            <li class="page-item"><a class="page-link" href="#">1</a></li>
            <li class="page-item"><a class="page-link" href="#">2</a></li>
            <li class="page-item"><a class="page-link" href="#">3</a></li>
            <li class="page-item"><a class="page-link" href="#">Next</a></li>
          </ul>
        </footer>
      </main>
    </div>

    <!-- Form thêm mới nhân viên -->
    <div v-if="activateForm" class="overlay" >
      <form class="form">
        <div class="d-flex justify-content-between align-items-center">
          <h4>{{checkStatusForm=='add'?'Thêm mới nhân viên':'Chỉnh sửa nhân viên'}}</h4>
          <i class="fa-solid fa-xmark" @click='handleCloseForm'></i>
        </div>
        <div>
          <label class="form-label" for="userName">Họ và tên</label>
          <input  v-model="initialUser.name" id="userName" type="text" class="form-control"  />
          <div class="form-text error" v-if="initialError.name!=''">{{initialError.name}}</div>
        </div>
        <div>
          <label class="form-label" for="dateOfBirth">Ngày sinh</label>
          <input  v-model="initialUser.birthday" id="dateOfBirth"  type="date" class="form-control" />
        </div>
        <div class="form-text error" v-if="initialError.birthday!=''" >{{initialError.birthday}}</div>
        <div>
          <label  class="form-label" for="email">Email</label>
          <input  v-model="initialUser.email" id="email" type="text" class="form-control" />
        </div>
        <div class="form-text error" v-if="initialError.email!=''" >{{initialError.email}}</div>
        <div>
          <label class="form-label" for="address">Địa chỉ</label>
          <textarea  v-model="initialUser.address" class="form-control" id="address" rows="3"></textarea>
        </div>
        <div>
          <button class="w-100 btn btn-primary" @click.prevent="handleSubmit">Thêm mới</button>
        </div>
      </form>
    </div>
    <!-- Modal xác nhận chặn tài khoản -->
    <div class="overlay" hidden>
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn chặn tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light">Hủy</button>
          <button class="btn btn-danger">Xác nhận</button>
        </div>
      </div>
    </div>

    <!-- Modal xác nhận xóa tài khoản -->
    <div class="overlay" v-if="deleteModal">
      <div class="modal-custom">
        <div class="modal-title">
          <h4>Cảnh báo</h4>
          <i class="fa-solid fa-xmark"></i>
        </div>
        <div class="modal-body-custom">
          <span>Bạn có chắc chắn muốn xóa tài khoản này?</span>
        </div>
        <div class="modal-footer-custom">
          <button class="btn btn-light" @click="handleCloseDelete">Hủy</button>
          <button class="btn btn-danger" @click="handleDelete()">Xác nhận</button>
        </div>
      </div>
    </div>
  </body>

</template>
<style scoped>
</style>