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
            <table class="w-full border border-gray-200 rounded-lg overflow-hidden shadow-sm">
             <thead class="bg-gray-100 text-gray-700 uppercase text-sm">
          <tr class="bg-gray-100 text-center hover:bg-gray-50">
            <th class="border border-gray-300 px-4 py-2">Id</th>
            <th class="border border-gray-300 px-4 py-2">Image</th>
            <th class="border border-gray-300 px-4 py-2">Name</th>
            <th class="border border-gray-300 px-4 py-2">Price</th>
            <th class="border border-gray-300 px-4 py-2">Manage</th>

          </tr>
        </thead>
        <tbody class="divide-y divide-gray-200">
  <tr
    v-for="product in products"
    :key="product.id"
    class="border-b border-gray-200 hover:bg-gray-50"
  >
    <!-- ID -->
    <td class="border px-4 py-3 text-center border-gray-300">
      {{ product.id }}
    </td>

    <!-- IMAGE -->
 <td class="px-4 py-3 border border-gray-300">
  <div class=" overflow-hidden rounded-xl">
    <img
      :src="product.image_url"
      class="w-48 h-48 object-contain "
    />
  </div>
</td>

    <!-- NAME -->
    <td class="border border-gray-300 px-4 py-3 font-medium text-gray-900">
      {{ product.name }}
    </td>

  

    <!-- PRICE -->
    <td class="border border-gray-300 px-4 py-3 font-semibold text-teal-500">
      {{ product.price }}$
    </td>

   

    <!-- ACTIONS -->
    <td class="border border-gray-300 px-4 py-3">
      <div class="flex items-center gap-2">
        <button
          @click="openProductModal(product)"
          class="rounded-lg border border-gray-300 p-2 hover:bg-gray-100"
        >
          ✏️
        </button>
        <button
          @click="openDetailModal(product)"
         class="rounded-lg border border-blue-300 p-2 text-blue-500 hover:bg-blue-50"
        >
  👁️
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
  <!-- Modal edit + add-->
      <Teleport to="body">
  <div
    v-if="isModalOpen"
    @click.self="closeModal"
    class="fixed inset-0 z-[99999] bg-black/40 backdrop-blur-sm flex justify-center items-center"
  >
    <div class="bg-white p-6 rounded shadow-lg w-full max-w-2xl max-h-[90vh] overflow-y-auto scrollbar-thin">
      
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
              <label for="name" class="block text-sm font-semibold">Product Description</label>
              <input
                type="text"
                id="description"
                v-model="form.description"
                class="w-full p-2 border rounded"
                placeholder="Product Description"
                required
              />
            </div>
            <div class="mb-4">
              <label for="price" class="block text-sm font-semibold">Product Price</label>
              <input
                type="number"
                id="price"
                step="0.01"
                v-model="form.price"
                class="w-full p-2 border rounded"
                placeholder="Product Price"
                required
              />
            </div>

            <div class="mb-4">
              <label for="img_url" class=" block text-sm font-semibold">Image URL</label>
               <img
     v-if="previewImage"
  :src="previewImage"
  class="object-cover rounded mb-3 w-48 h-48"
  />

              <input
                type="file"
                id="img_url"
                @change="handleFileChange"
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
</Teleport>
<!-- Modal detail product -->
<Teleport to="body">
  <div
    v-if="isDetailModalOpen"
    @click.self="closeDetailModal"
    class="fixed inset-0 z-[99999] bg-black/40 flex justify-center items-center"
  >
    <div class="bg-white p-6 rounded w-full max-w-2xl">

      <h2 class="text-xl font-bold mb-4">Product Detail</h2>

      <div v-if="selectedProduct" class="space-y-3">

        <img
          :src="'http://127.0.0.1:8000/storage/' + selectedProduct.img_url"
          class="w-48 h-48 object-cover rounded"
        />

        <p><b>ID:</b> {{ selectedProduct.id }}</p>
        <p><b>Name:</b> {{ selectedProduct.name }}</p>
        <p><b>Description:</b> {{ selectedProduct.description }}</p>
        <p><b>Price:</b> {{ selectedProduct.price }}$</p>

        <p><b>Category:</b> {{ selectedProduct.category?.name }}</p>
        <p><b>Brand:</b> {{ selectedProduct.brand?.name }}</p>

        <p><b>Created:</b> {{ selectedProduct.created_at }}</p>
        <p><b>Updated:</b> {{ selectedProduct.updated_at }}</p>

      </div>

      <div class="flex justify-end mt-4">
        <button
          @click="closeDetailModal"
          class="bg-gray-500 text-white px-4 py-2 rounded"
        >
          Close
        </button>
      </div>

    </div>
  </div>
</Teleport>
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
      isDetailModalOpen: false,
      selectedProduct: null,    
      previewImage: null,
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
  handleFileChange(e) {
  const file = e.target.files[0];

  if (!file) return;

  this.form.img_url = file;

  // 🔥 QUAN TRỌNG: tạo preview đúng cách
  this.previewImage = URL.createObjectURL(file);
},
    async handleProductSubmit() {
   
  try {
    const data = new FormData();

    data.append("name", this.form.name);
    data.append("description", this.form.description);

    data.append("price", this.form.price);
    data.append("category_id", this.form.category_id);
    data.append("brand_id", this.form.brand_id);

    if (this.form.img_url ) {
      data.append("img_url", this.form.img_url);
    }

    if (this.isEditing) {
      data.append("_method", "PUT");

      await axios.post(
        `http://127.0.0.1:8000/api/products/${this.form.id}`,
        data
      );
    } else {
      await axios.post("http://127.0.0.1:8000/api/products", data);
    }

    await this.fetchProducts();
    this.closeModal();
  } catch (error) {
    console.error(error);
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
    
   
     openDetailModal(product) {
  this.selectedProduct = product;
  this.isDetailModalOpen = true;
},
closeDetailModal() {
  this.isDetailModalOpen = false;
  this.selectedProduct = null;
},
   
  openProductModal(product = null) {
  this.isEditing = !!product;

  if (product) {
    // 👉 FORM chỉ chứa dữ liệu text
    this.form = {
      id: product.id,
      name: product.name,
      description: product.description,
      price: product.price,
      category_id: product.category_id,
      brand_id: product.brand_id,
      img_url: null, // reset file
    };

    // 👉 ảnh cũ để preview
     this.previewImage =
      product.img_url
        ? "http://127.0.0.1:8000/storage/" + product.img_url
        : null;

  } else {
    this.form = {
      id: null,
      name: "",
      description: "",
      price: "",
      category_id: null,
      brand_id: null,
      img_url: null,
    };

    this.previewImage = null;
  }

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
