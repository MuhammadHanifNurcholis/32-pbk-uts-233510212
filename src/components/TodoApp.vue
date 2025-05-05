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
// Tambahkan fungsi untuk edit dan delete
const editingIndex = ref(null);
const editingText = ref("");
const editingPrice = ref(0);

const removeItem = (index) => {
  items.value.splice(index, 1);
  if (editingIndex.value === index) cancelEdit();
};

const editItem = (index) => {
  editingIndex.value = index;
  editingText.value = items.value[index].text;
  editingPrice.value = items.value[index].price;
};

const saveItem = (index) => {
  const name = editingText.value.trim();
  const price = parseFloat(editingPrice.value);
  if (!name) {
    alert("Nama tidak boleh kosong.");
    return;
  }
  if (isNaN(price) || price <= 0) {
    alert("Harga tidak valid.");
    return;
  }
  items.value[index].text = name;
  items.value[index].price = price;
  cancelEdit();
};

const cancelEdit = () => {
  editingIndex.value = null;
  editingText.value = "";
  editingPrice.value = 0;
};

// Tambahkan computed total price
const totalPrice = computed(() => {
  return items.value.reduce((sum, item) => sum + (item.price || 0), 0);
});

// Tambahkan local storage functionality
onMounted(() => {
  const saved = localStorage.getItem("shopping-items");
  if (saved) {
    try {
      items.value = JSON.parse(saved);
    } catch (e) {
      console.error("Failed to parse saved data:", e);
    }
  }
});

watch(
  items,
  (newVal) => {
    localStorage.setItem("shopping-items", JSON.stringify(newVal));
  },
  { deep: true }
);
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
<!-- Update template untuk edit/delete -->
<template v-if="editingIndex === index">
  <input
    v-model="editingText"
    class="edit-input"
    @keyup.enter="saveItem(index)"
    @blur="saveItem(index)"
  />
  <input
    v-model.number="editingPrice"
    type="number"
    class="edit-input"
    placeholder="Harga"
  />
</template>

<div class="task-actions">
  <template v-if="editingIndex === index">
    <button @click="saveItem(index)" class="btn btn-save">💾</button>
    <button @click="cancelEdit" class="btn btn-cancel">✖</button>
  </template>
  <template v-else>
    <button @click="editItem(index)" class="btn btn-edit">✏️</button>
    <button @click="removeItem(index)" class="btn btn-remove">🗑</button>
  </template>
</div>
<!-- Tambahkan filter controls -->
<div class="filter-section">
  <label class="filter-label">
    <input type="checkbox" v-model="showOnlyUnpurchased" />
    Tampilkan hanya yang belum dibeli
  </label>
  <select v-model="sortOption" class="input-task" style="max-width: 200px">
    <option value="name">Urutkan: Nama</option>
    <option value="price-asc">Harga Termurah</option>
    <option value="price-desc">Harga Termahal</option>
  </select>
</div>

<!-- Tambahkan total section -->
<div class="total-section">
  Total: <span>Rp {{ totalPrice.toLocaleString() }}</span><br />
  Jumlah Barang: <span>{{ items.length }}</span>
</div>

</template>

<style>
</style>