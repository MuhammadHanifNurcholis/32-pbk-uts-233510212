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

// Tambahkan computed properties untuk sorting dan filtering
const showOnlyUnpurchased = ref(false);
const sortOption = ref("name");

const sortedItems = computed(() => {
  const sorted = [...items.value];
  switch (sortOption.value) {
    case "price-asc":
      return sorted.sort((a, b) => a.price - b.price);
    case "price-desc":
      return sorted.sort((a, b) => b.price - a.price);
    default:
      return sorted.sort((a, b) => a.text.localeCompare(b.text));
  }
});

const filteredItems = computed(() => {
  return showOnlyUnpurchased.value
    ? sortedItems.value.filter((item) => !item.purchased)
    : sortedItems.value;
});
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
<!-- Tambahkan list item -->
<div class="task-list-wrapper">
  <ul class="task-list">
    <li v-for="(item, index) in filteredItems" :key="index" class="task-item">
      <div class="task-label">
        <input type="checkbox" v-model="item.purchased" />
        <span :class="{ completed: item.purchased }">{{ item.text }}</span>
        <span class="price">Rp {{ item.price.toLocaleString() }}</span>
      </div>
    </li>
  </ul>
</div>
</template>

<style>
</style>