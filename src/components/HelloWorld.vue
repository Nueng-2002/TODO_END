<template>
  <div class="Bg-all">
    <div class="container">
      <h1 class="header">เมนูอาหารที่คุณต้องการเพิ่ม</h1>

      <div class="input-container">
        <input class="input-field" @keyup.enter="addMenuItem" v-model="newFood.name" type="text"
          placeholder="ชื่ออาหาร" />
        <input class="input-field" @keyup.enter="addMenuItem" v-model="newFood.price" type="number" placeholder="ราคา" />
        <input class="input-field" @keyup.enter="addMenuItem" v-model="newFood.comment" type="text"
          placeholder="คำแนะนำ" />
        <label class="file-label">
          เลือกรูปภาพ
          <input type="file" @change="onFileChange" class="file-input" />
        </label>
        <button class="add-button" @click="addMenuItem">เพิ่ม</button>
      </div>

      <div class="menu-section">
        <div class="menu-list">
          <h1 class="menu-header">รายการอาหารที่เพิ่มเข้ามา</h1>
          <div v-for="(food, index) in menu" :key="index" class="menu-item">
            <div class="food-info">
              <img class="food-image" v-if="food.imageUrl" :src="food.imageUrl" alt="Preview" />
              <div class="food-details">
                <div>ชื่ออาหาร: {{ food.name }}</div>
                <div>ราคา: {{ food.price }} บาท</div>
                <div>แนะนำ: {{ food.comment }}</div>
                <label class="checkbox_01">
                  <input type="checkbox" v-model="food.checked" @change="updateSelectedFoods(food)" />
                  เลือกอาหาร
                </label>
              </div>
            </div>
            <div class="menu-actions">
              <button class="delete-button" @click="deleteMenuItem(index)">
                ลบ
              </button>
              <button class="edit-button" @click="editFood(index)">
                แก้ไข
              </button>
            </div>
          </div>
        </div>

        <div class="selected-foods">
          <h2>อาหารที่เลือก</h2>
          <div v-for="(food, index) in selectedFoods" :key="index" class="selected-item">
            <div class="Food_item">
              ชื่อ {{ food.name }} - ราคา {{ food.price }} บาท
            </div>
            <button class="removeSelected_Button" @click="removeSelected(food)">
              เอาออก
            </button>
          </div>

          <div class="total">
            <h3>ยอดรวม: {{ totalPrice }} บาท</h3>
          </div>

          <button class="payment_Button" @click="processPayment">
            ชำระเงิน
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "RestaurantMenu",
  data() {
    return {
      menu: [], // รายการอาหารทั้งหมด
      newFood: {
        // ข้อมูลอาหารใหม่ที่เพิ่มเข้ามา
        name: "",
        price: null,
        comment: "",
        // เก็บข้อมูล //
        imageUrl: null, // เก็บ URL ของภาพ
        checked: false, // สถานะการเลือกอาหาร
      },
      selectedFoods: [], // รายการอาหารที่เลือก
      isEditing: false, // สถานะการแก้ไขอาหาร
      editIndex: null, // ดัชนีของอาหารที่กำลังแก้ไข
    };
  },
  computed: {
    totalPrice() {
      // คำนวณยอดรวมของอาหารที่เลือก
      return this.selectedFoods.reduce(
        (total, food) => total + (parseFloat(food.price) || 0), 0);
    },
  },
  methods: {
    onFileChange(event) {
      // ดึงไฟล์แรกจากการเลือกไฟล์
      const file = event.target.files[0];

      // สร้างตัวอ่านไฟล์
      const reader = new FileReader();

      // เมื่ออ่านไฟล์เสร็จแล้ว
      reader.onload = (e) => {
        // เก็บ URL ของภาพใน newFood.imageUrl
        this.newFood.imageUrl = e.target.result;
      };

      // ตรวจสอบว่ามีไฟล์หรือไม่
      if (file) {
        // เริ่มอ่านไฟล์เป็น Data URL
        reader.readAsDataURL(file);
      }
    },
    addMenuItem() {
      // ตรวจสอบข้อมูลก่อนเพิ่ม
      if (
        !this.newFood.name ||
        !this.newFood.price ||
        !this.newFood.comment ||
        !this.newFood.imageUrl
      ) {
        alert("กรุณากรอกข้อมูลให้ครบถ้วน");
        return;
      }

      if (this.isEditing) {
        // แก้ไขอาหารที่เลือก
        this.menu[this.editIndex] = this.newFood;
        this.isEditing = false;
      } else {
        // เพิ่มอาหารใหม่
        this.menu.push(this.newFood);
      }

      // รีเซ็ตข้อมูลสำหรับเพิ่มอาหารใหม่
      this.resetNewFood();
    },
    resetNewFood() {
      this.newFood = {
        name: "",
        price: null,
        comment: "",
        imageUrl: null,
        checked: false,
      };
    },
    deleteMenuItem(index) {
      // ลบอาหารจากรายการเมนู
      this.menu.splice(index, 1);
    },
    editFood(index) {
      // ตั้งค่าอาหารที่ต้องการแก้ไข
      this.newFood = this.menu[index]; // คัดลอกข้อมูล
      this.isEditing = true;
      this.editIndex = index; // เก็บดัชนีสำหรับการแก้ไข
    },
    updateSelectedFoods(food) {
      // อัปเดตรายการอาหารที่เลือก
      if (food.checked) {
        this.selectedFoods.push(food);
      } else {
        this.selectedFoods = this.selectedFoods.filter(
          (selected) => selected.name !== food.name
        );
      }
    },
    removeSelected(food) {
      // ลบอาหารจากรายการที่เลือก
      food.checked = false;
      this.selectedFoods = this.selectedFoods.filter(
        (selected) => selected.name !== food.name
      );
    },
    processPayment() {
      // ฟังก์ชันสำหรับจัดการการชำระเงิน
      alert(`ยอดรวมของคุณคือ ${this.totalPrice} บาท`);
      // คุณสามารถเพิ่มลอจิกการชำระเงินที่นี่
    },
  },
};
</script>
