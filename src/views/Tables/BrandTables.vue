<template>
  <AdminLayout>
    <PageBreadcrumb :pageTitle="currentPageTitle" />
    <div class="space-y-5 sm:space-y-6">
   
        <BrandTable />
     
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref } from "vue";
import PageBreadcrumb from "@/components/common/PageBreadcrumb.vue";
import AdminLayout from "@/components/layout/AdminLayout.vue";
// import ComponentCard from "@/components/common/ComponentCard.vue";
import BrandTable from "@/components/tables/basic-tables/BrandTable.vue";
const currentPageTitle = ref("Brand Tables");
</script>
