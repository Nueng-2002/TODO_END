<template>
  <div class="Bg-all">
    <div class="container">
      <h1 class="header">เมนูอาหารที่คุณต้องการเพิ่ม</h1>

      <div class="input-container">
        <input class="input-field" @keyup.enter="addMenuItem" v-model="newFood.name" type="text"
          placeholder="ชื่ออาหาร" />
        <input class="input-field" @keyup.enter="addMenuItem" v-model="newFood.price" type="number" placeholder="ราคา" />
        <textarea class="input-field" @keyup.enter="addMenuItem" v-model="newFood.comment" placeholder="คำแนะนำ"
          rows="3"></textarea>
        <label class="file-label">
          เลือกรูปภาพ
          <input type="file" @change="onFileChange" class="file-input" />
        </label>
        <button class="add-button" @click="addMenuItem">
          {{ isEditing ? "อัพเดต" : "เพิ่ม" }}
        </button>
      </div>

      <div class="menu-section">
        <div class="menu-list">
          <h1 class="menu-header">รายการอาหารที่เพิ่มเข้ามา</h1>
          <div v-for="(food, index) in menu" :key="index" class="menu-item">
            <div class="food-info">
              <img class="food-image" v-if="food.image" :src="food.image" alt="Preview" />
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
              <button class="delete-button" @click="deleteMenu(food.id)">
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
import Axios from "axios";

export default {
  name: "RestaurantMenu",
  data() {
    return {
      menu: [],
      newFood: {
        name: "",
        price: null,
        comment: "",
        image: null,
        checked: false,
        id: null, // เพิ่ม id ใน newFood
      },
      selectedFoods: [],
      isEditing: false,
      editIndex: null,
    };
  },
  computed: {
    //-------------------------- ผลรวมเงินทั้งหมด ------------------------------//
    totalPrice() {
      return this.selectedFoods.reduce(
        (total, food) => total + (parseFloat(food.price) || 0),
        0
      );
    },
  },
  methods: {
    //-------------------------- รูปภาพ ------------------------------//
    onFileChange(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (e) => {
        this.newFood.image = e.target.result;
      };

      if (file) {
        reader.readAsDataURL(file);
      }
    },

    async getData() {
      try {
        const res = await Axios.get("http://127.0.0.1:1111", {});
        this.menu = res.data;
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },

    async addMenuItem() {
      const { name, price, comment, image } = this.newFood;
      if (!name || !price || !comment || !image) {
        alert("กรุณากรอกข้อมูลให้ครบถ้วน");
        return; // หยุดการทำงานหากข้อมูลไม่ครบ
      }

      const body = { name, price, comment, image };

      try {
        if (this.isEditing) {
          // อัพเดตข้อมูลเมื่อกำลังแก้ไข
          await Axios.put(`http://127.0.0.1:1111/update/${this.newFood.id}`, body);
          this.menu.splice(this.editIndex, 1, { ...body, id: this.newFood.id });
          this.isEditing = false; // เปลี่ยนสถานะการแก้ไข
          await this.getData();
        } else {

          // เพิ่มเมนูใหม่
          await Axios.post("http://127.0.0.1:1111/create/", body);
          this.menu.push(this.newFood);
          await this.getData();
        }
      } catch (error) {
        console.error("เกิดข้อผิดพลาดในการเพิ่มเมนู:", error);
        alert("ไม่สามารถเพิ่มเมนูได้");
      }
    },

    async deleteMenu(id) {
      console.log(id)
      await Axios.delete(`http://127.0.0.1:1111/Delete/${id}`);
      await this.getData();
    },

    updateSelectedFoods(food) {
      if (food.checked) {
        if (!this.selectedFoods.find((selected) => selected.id === food.id)) {
          this.selectedFoods.push(food);
        }
      } else {
        this.selectedFoods = this.selectedFoods.filter(
          (selected) => selected.id !== food.id
        );
      }
    },

    resetNewFood() {
      this.newFood = {
        name: "",
        price: null,
        comment: "",
        image: null,
        checked: false,
        id: null, // รีเซ็ต id ด้วย
      };
    },
    editFood(index) {
      const foodToEdit = this.menu[index];
      this.newFood = { ...foodToEdit }; // คัดลอกข้อมูล
      this.isEditing = true;
      this.editIndex = index;
    },

    updateSelectedFoods(food) {
      if (food.checked) {
        this.selectedFoods.push(food);
      } else {
        this.selectedFoods = this.selectedFoods.filter(
          (selected) => selected.name !== food.name
        );
      }
    },

    removeSelected(food) {
      food.checked = false;
      this.selectedFoods = this.selectedFoods.filter(
        (selected) => selected.name !== food.name
      );
    },

    processPayment() {
      alert(`ยอดรวมของคุณคือ ${this.totalPrice} บาท`);
      // คุณสามารถเพิ่มลอจิกการชำระเงินที่นี่
    },
  },
  mounted() {
    this.getData();
  },
};
</script>
