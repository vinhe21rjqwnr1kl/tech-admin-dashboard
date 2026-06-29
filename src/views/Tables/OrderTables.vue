<template>
  <AdminLayout>
    <PageBreadcrumb :pageTitle="currentPageTitle" />
    <div class="space-y-5 sm:space-y-6">
        <OrderTable />
    </div>
  </AdminLayout>
</template>

<script setup>
import { ref } from "vue";
import PageBreadcrumb from "@/components/common/PageBreadcrumb.vue";
import AdminLayout from "@/components/layout/AdminLayout.vue";
import OrderTable from "@/components/tables/basic-tables/OrderTable.vue";
// import ComponentCard from "@/components/common/ComponentCard.vue";
const currentPageTitle = ref("Category Tables");
</script>
