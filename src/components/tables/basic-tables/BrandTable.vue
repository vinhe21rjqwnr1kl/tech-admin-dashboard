<template>
  <div>
    <h1 style="text-align: center;font-size: 20px;">Manage Brands</h1>

    <div v-if="loading" class="text-center text-blue-600">
      Loading brands...
    </div>

    <div v-else>
      <button
        @click="openModal(null)"
        class="bg-teal-500 text-white px-4 py-2 rounded mb-4"
      >
        Add Brand
      </button>

      <table class="table-auto w-full border-collapse border border-gray-200">
        <thead>
          <tr class="bg-gray-100 text-center">
            <th class="border border-gray-300 px-4 py-2">Id</th>
            <th class="border border-gray-300 px-4 py-2">Name</th>
            <th class="border border-gray-300 px-4 py-2">Description</th>
            <th class="border border-gray-300 px-4 py-2">Manage</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="brand in brands" :key="brand.id">
            <td class="border border-gray-300 px-4 py-2 text-center">
              {{ brand.id }}
            </td>

            <td class="border border-gray-300 px-4 py-2 text-center">
              {{ brand.name }}
            </td>

            <td class="border border-gray-300 px-4 py-2 text-center">
              {{ brand.description }}
            </td>

            <td class="border border-gray-300 px-4 py-2 text-center">
              <button
                @click="openModal(brand)"
                class="rounded-lg border border-gray-300 p-2 hover:bg-gray-100"
              >
                ✏️
              </button>

              <button
                @click="deleteBrand(brand.id)"
                class="rounded-lg border border-red-300 p-2 text-red-500 hover:bg-red-50"
              >
                🗑️
              </button>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- MODAL -->
      <div
        v-if="isModalOpen"
        @click.self="closeModal"
        class="fixed inset-0 bg-black/40 backdrop-blur-sm flex justify-center items-center"
      >
        <div class="bg-white text-gray-900 p-6 rounded shadow-lg w-1/3">
          <h2 class="text-xl font-semibold mb-4">
            {{ isEditing ? "Edit Brand" : "Add Brand" }}
          </h2>

          <form @submit.prevent="handleSubmit">
            <div class="mb-4">
              <label class="block text-sm font-semibold">Name</label>
              <input
                v-model="form.name"
                class="w-full p-2 border border-gray-300 rounded"
                required
              />
            </div>

            <div class="mb-4">
              <label class="block text-sm font-semibold">Description</label>
              <input
                v-model="form.description"
                class="w-full p-2 border border-gray-300 rounded"
              />
            </div>

            <div class="flex justify-end">
              <button
                type="button"
                @click="closeModal"
                class="mr-4 text-gray-500"
              >
                Cancel
              </button>

              <button class="bg-teal-500 text-white px-4 py-2 rounded">
                {{ isEditing ? "Save Changes" : "Add Brand" }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import { ref, onMounted } from "vue";

interface Brand {
  id: number;
  name: string;
  description: string;
}

const brands = ref<Brand[]>([]);
const loading = ref<boolean>(true);
const error = ref<string | null>(null);

const isModalOpen = ref<boolean>(false);
const isEditing = ref<boolean>(false);

const form = ref<Brand>({
  id: 0,
  name: "",
  description: "",
});

const fetchBrands = async (): Promise<void> => {
  try {
    const res = await axios.get<Brand[]>(
      "http://127.0.0.1:8000/api/brands"
    );
    brands.value = res.data;
  } catch (err) {
    console.error(err);
    error.value = "Failed to load brands";
  } finally {
    loading.value = false;
  }
};

const openModal = (brand: Brand | null): void => {
  if (brand) {
    isEditing.value = true;
    form.value = { ...brand };
  } else {
    isEditing.value = false;
    form.value = { id: 0, name: "", description: "" };
  }

  isModalOpen.value = true;
};

const closeModal = (): void => {
  isModalOpen.value = false;
};

const handleSubmit = async (): Promise<void> => {
  if (isEditing.value) {
    await updateBrand();
  } else {
    await addBrand();
  }

  closeModal();
};

const addBrand = async (): Promise<void> => {
  const res = await axios.post(
    "http://127.0.0.1:8000/api/brands",
    form.value
  );

  brands.value.push(res.data);
};

const updateBrand = async (): Promise<void> => {
  const res = await axios.put(
    `http://127.0.0.1:8000/api/brands/${form.value.id}`,
    form.value
  );

  const index = brands.value.findIndex(
    (b) => b.id === form.value.id
  );

  if (index !== -1) {
    brands.value[index] = res.data;
  }
};

const deleteBrand = async (id: number): Promise<void> => {
  if (!confirm("Are you sure?")) return;

  await axios.delete(`http://127.0.0.1:8000/api/brands/${id}`);

  brands.value = brands.value.filter((b) => b.id !== id);
};

onMounted(fetchBrands);
</script>
<style lang="scss" scoped>

</style>