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
:root {
  --primary-color: #10b981;
  --primary-hover: #059669;
  --danger-color: #ef4444;
  --danger-hover: #dc2626;
  --bg-color: #f3f4f6;
  --card-color: #ffffff;
  --text-color: #1f2937;
  --completed-color: red;
  --shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: "Inter", sans-serif;
  color: var(--text-color);
  overflow: hidden;
  background-color: var(--bg-color);
}

#app {
  height: 100%;
}

.app-background {
  height: 100vh;
  width: 100vw;
  background: linear-gradient(135deg, #34d399, #059669);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  box-sizing: border-box;
}

.app-container {
  width: 100%;
  max-width: 600px;
  max-height: 90vh;
  background-color: var(--card-color);
  border-radius: 1rem;
  box-shadow: var(--shadow);
  padding: 1.5rem;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.title {
  text-align: center;
  font-size: 1.75rem;
  font-weight: bold;
  margin-bottom: 1rem;
  color: var(--primary-color);
}

.form-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

@media (min-width: 500px) {
  .form-section {
    flex-direction: row;
  }
}

.input-task,
.input-price {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 1px solid #d1d5db;
  border-radius: 0.75rem;
  font-size: 1rem;
}

.input-price {
  max-width: 150px;
}

.btn {
  padding: 0.5rem 0.75rem;
  border: none;
  border-radius: 0.5rem;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.2s ease;
  font-size: 1rem;
}

.btn-add {
  background-color: var(--primary-color);
  color: white;
}

.btn-add:hover {
  background-color: var(--primary-hover);
}

.btn-remove {
  background-color: var(--danger-color);
  color: white;
}

.btn-remove:hover {
  background-color: var(--danger-hover);
}

.btn-edit {
  background-color: #f59e0b;
  color: white;
}

.btn-edit:hover {
  background-color: #d97706;
}

.btn-save {
  background-color: #3b82f6;
  color: white;
}

.btn-save:hover {
  background-color: #2563eb;
}

.btn-cancel {
  background-color: #6b7280;
  color: white;
}

.btn-cancel:hover {
  background-color: #4b5563;
}

.filter-section {
  text-align: center;
  margin-bottom: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
}

@media (min-width: 500px) {
  .filter-section {
    flex-direction: row;
    justify-content: space-between;
  }
}

.filter-label {
  font-size: 0.95rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.task-list-wrapper {
  flex: 1;
  overflow-y: auto;
  scrollbar-width: none;
}

.task-list-wrapper::-webkit-scrollbar {
  display: none;
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-item {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: flex-start;
  padding: 0.75rem 1rem;
  margin-bottom: 0.5rem;
  background-color: #f9fafb;
  border-radius: 0.75rem;
  border: 1px solid #e5e7eb;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
}

@media (min-width: 500px) {
  .task-item {
    flex-direction: row;
    align-items: center;
  }
}

.task-label {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex: 1;
  flex-wrap: wrap;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 0.5rem;
}

@media (min-width: 500px) {
  .task-actions {
    margin-top: 0;
  }
}

.task-item input[type="checkbox"] {
  transform: scale(1.2);
}

.task-item span {
  font-size: 1rem;
  flex: 1;
}

.edit-input {
  flex: 1;
  padding: 0.5rem;
  font-size: 1rem;
  border-radius: 0.5rem;
  border: 1px solid #d1d5db;
  min-width: 100px;
}

.price {
  font-size: 0.95rem;
  color: #374151;
  margin-left: 1rem;
}

.completed {
  text-decoration: line-through;
  color: var(--completed-color);
}

.total-section {
  text-align: right;
  font-size: 1.1rem;
  font-weight: bold;
  padding-top: 1rem;
  border-top: 1px solid #e5e7eb;
  margin-top: 1rem;
  color: var(--text-color);
}

</style>