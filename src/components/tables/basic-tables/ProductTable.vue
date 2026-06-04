<template>
  <div>
    <div class="container mx-auto p-6">
      <h1 class="text-3xl font-bold text-gray-800 mb-6">Products</h1>
      <!-- Add Product Button -->
      <button
        @click="openProductModal()"
        class="bg-teal-500 text-dark px-4 py-2 rounded mb-4"
      >
        Add Product
      </button>
    
      <div v-if="loading" class="text-center text-blue-600">Loading products...</div>
      <div v-else>
            <table class="table-auto w-full border-collapse border border-gray-200">
             <thead>
          <tr class="bg-gray-100 text-center">
            <th class="border border-gray-300 px-4 py-2">Id</th>
            <th class="border border-gray-300 px-4 py-2">Image</th>
            <th class="border border-gray-300 px-4 py-2">Name</th>
            <th class="border border-gray-300 px-4 py-2">Description</th>
            <th class="border border-gray-300 px-4 py-2">Price</th>
            <th class="border border-gray-300 px-4 py-2">Category Name</th>
            <th class="border border-gray-300 px-4 py-2">Brand Name</th>
            <th class="border border-gray-300 px-4 py-2">Manage</th>

          </tr>
        </thead>
        <tbody>
  <tr
    v-for="product in products"
    :key="product.id"
    class="border-b border-gray-200 hover:bg-gray-50"
  >
    <!-- ID -->
    <td class="px-4 py-3 text-center">
      {{ product.id }}
    </td>

    <!-- IMAGE -->
    <td class="px-4 py-3">
      <img
        :src="product.img_url"
        class="w-12 h-12 object-cover rounded"
      />
    </td>

    <!-- NAME -->
    <td class="px-4 py-3 font-medium text-gray-900">
      {{ product.name }}
    </td>

    <!-- DESCRIPTION -->
    <td class="px-4 py-3 text-sm text-gray-600">
      {{ product.description }}
    </td>

    <!-- PRICE -->
    <td class="px-4 py-3 font-semibold text-teal-500">
      {{ product.price }}$
    </td>

    <!-- CATEGORY -->
    <td class="px-4 py-3 text-sm text-gray-600">
      {{ product.category?.name }}
    </td>

    <!-- BRAND -->
    <td class="px-4 py-3 text-sm text-gray-600">
      {{ product.brand?.name }}
    </td>

    <!-- ACTIONS -->
    <td class="px-4 py-3">
      <div class="flex items-center gap-2">
        <button
          @click="openProductModal(product)"
          class="rounded-lg border border-gray-300 p-2 hover:bg-gray-100"
        >
          ✏️
        </button>

        <button
          @click="deleteProduct(product.id)"
          class="rounded-lg border border-red-300 p-2 text-red-500 hover:bg-red-50"
        >
          🗑️
        </button>
      </div>
    </td>
  </tr>
</tbody>

            </table>
       
      </div>
  
      <!-- Product Modal -->
      <div
        v-if="isModalOpen"
        @click.self="closeModal"
        class="fixed inset-0 bg-black/40 backdrop-blur-sm flex justify-center items-center"
      >
        <div class="bg-white p-6 rounded shadow-lg w-1/3">
          <h2 class="text-xl font-semibold mb-4">
            {{ isEditing ? "Edit Product" : "Add Product" }}
          </h2>
          <form @submit.prevent="handleProductSubmit">
            <div class="mb-4">
              <label for="name" class="block text-sm font-semibold">Product Name</label>
              <input
                type="text"
                id="name"
                v-model="form.name"
                class="w-full p-2 border rounded"
                placeholder="Product Name"
                required
              />
            </div>

            <div class="mb-4">
              <label for="price" class="block text-sm font-semibold">Product Price</label>
              <input
                type="number"
                id="price"
                v-model="form.price"
                class="w-full p-2 border rounded"
                placeholder="Product Price"
                required
              />
            </div>

            <div class="mb-4">
              <label for="img_url" class="block text-sm font-semibold">Image URL</label>
              <input
                type="text"
                id="img_url"
                v-model="form.img_url"
                class="w-full p-2 border rounded"
                placeholder="Image URL"
              />
            </div>

            <div class="mb-4">
              <label for="category" class="block text-sm font-semibold">Category</label>
              <select
                id="category"
                v-model="form.category_id"
                class="w-full p-2 border rounded"
                required
              >
                <option
                  v-for="category in categories"
                  :key="category.id"
                  :value="category.id"
                >
                  {{ category.name }}
                </option>
              </select>
            </div>

            <div class="mb-4">
              <label for="brand" class="block text-sm font-semibold">Brand</label>
              <select
                id="brand"
                v-model="form.brand_id"
                class="w-full p-2 border rounded"
                required
              >
                <option v-for="brand in brands" :key="brand.id" :value="brand.id">
                  {{ brand.name }}
                </option>
              </select>
            </div>

            <div class="flex justify-end">
              <button
                @click="closeModal"
                type="button"
                class="mr-4 bg-gray-500 text-white px-4 py-2 rounded"
              >
                Cancel
              </button>
              <button type="submit" class="bg-teal-500 text-dark px-4 py-2 rounded">
                {{ isEditing ? "Save Changes" : "Add Product" }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import axios from "axios";
export default {
 
  data() {
    return {
      products: [],
      categories: [],
      brands: [],
      cart: [],
      loading: true,
      isModalOpen: false,
      isEditing: false,
      form: {
        id: null,
        name: "",
        price: "",
        img_url: "",
        category_id: null,
        brand_id: null,
      },
    };
  },
  mounted() {
    this.fetchProducts();
    this.fetchCategories();
    this.fetchBrands();
    
  },
  methods: {
    async handleProductSubmit() {
      try {
        if (this.isEditing) {
          // Edit Product
          const response = await axios.put(
            `http://127.0.0.1:8000/api/products/${this.form.id}`,
            this.form
          );
          const updatedProduct = response.data;
          // Update the product in the local list
          const index = this.products.findIndex((p) => p.id === updatedProduct.id);
          if (index !== -1) {
            this.products[index] = updatedProduct;
          }
          this.fetchProducts();
        } else {
          // Add New Product
          const response = await axios.post(
            "http://127.0.0.1:8000/api/products",
            this.form
          );
          // const newProduct = response.data;
          // this.products.push(newProduct);
          await this.fetchProducts();
        }
        this.closeModal();
      } catch (error) {
        console.error("Error saving product:", error);
      }
    },
    async fetchProducts() {
      try {
        const response = await axios.get("http://127.0.0.1:8000/api/products");
        this.products = response.data;
        this.loading = false;
      } catch (error) {
        console.error("Error fetching products:", error);
      }
    },
    async deleteProduct(productId) {
      if (!confirm("Are you sure you want to delete this product?")) {
        return;
      }
      try {
        await axios.delete(`http://127.0.0.1:8000/api/products/${productId}`);
        this.products = this.products.filter((product) => product.id !== productId);
      } catch (error) {
        console.error("Error deleting product:", error);
      }
    },
    async fetchCategories() {
      try {
        const response = await axios.get("http://127.0.0.1:8000/api/categories");
        this.categories = response.data;
      } catch (error) {
        console.error("Error fetching categories:", error);
      }
    },
    async fetchBrands() {
      try {
        const response = await axios.get("http://127.0.0.1:8000/api/brands");
        this.brands = response.data;
      } catch (error) {
        console.error("Error fetching brands:", error);
      }
    },
    
   
   
   
    openProductModal(product = null) {
      this.isEditing = !!product;
      this.form = product
        ? { ...product }
        : {
            id: null,
            name: "",
            price: "",
            img_url: "",
            category_id: null,
            brand_id: null,
          };
      this.isModalOpen = true;
    },
    closeModal() {
      this.isModalOpen = false;
    },
    openCartModal() {
      this.isCartModalOpen = true;
    },
    closeCartModal() {
      this.isCartModalOpen = false;
    },
  },
};
</script>

<style lang="scss" scoped></style>
