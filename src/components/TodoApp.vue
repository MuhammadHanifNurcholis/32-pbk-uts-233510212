<script setup>
import { ref } from "vue";

const newItem = ref("");
const newPrice = ref(0);
const items = ref([]);

// Tambahkan fungsi addItem
const addItem = () => {
  const name = newItem.value.trim();
  const price = parseFloat(newPrice.value);
  if (!name) {
    alert("Nama barang tidak boleh kosong!");
    return;
  }
  if (isNaN(price) || price <= 0) {
    alert("Harga harus lebih dari 0!");
    return;
  }

  items.value.push({ text: name, price: price, purchased: false });
  newItem.value = "";
  newPrice.value = 0;
};
</script>

<template>
  <div class="app-background">
    <div class="app-container">
      <h1 class="title">🛒 Daftar Belanja Pribadi</h1>
    </div>
  </div>
  
  <!-- Tambahkan form input -->
<div class="form-section">
  <input
    v-model="newItem"
    @keyup.enter="addItem"
    placeholder="Nama barang..."
    class="input-task"
  />
  <input
    v-model.number="newPrice"
    type="number"
    placeholder="Harga"
    class="input-price"
  />
  <button @click="addItem" class="btn btn-add">➕ Tambah</button>
</div>
</template>

<style>
</style>