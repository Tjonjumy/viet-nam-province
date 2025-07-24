<template>
  <h2 class="mt-3 fw-bold">Tra cứu 3321 xã/phường mới của Việt Nam</h2>
  <div class="mt-3">
    <label class="label-txt">Tỉnh/Thành</label>
    <InputText
      v-model="province"
      v-on:keydown.enter.prevent='onSearch'
      name="province"
      type="text"
      placeholder="Tìm kiếm theo tỉnh/thành"
      class="inp-search"
    />
  </div>
  <div class="mt-3">
    <label class="label-txt">Xã/Phường</label>
    <InputText
      v-model="ward"
      v-on:keydown.enter.prevent='onSearch'
      name="ward"
      type="text"
      placeholder="Tìm kiếm theo xã/phường"
      class="inp-search"
    />
  </div>
  <div class="btn-search mt-3">
    <Button label="Tìm kiếm" icon="pi pi-search" @click="onSearch" />
  </div>
  <DataTable
    :value="datas"
    sortField="id"
    :sortOrder="1"
    scrollable
    scrollHeight="400px"
    paginator
    :rows="5"
    :rowsPerPageOptions="[5, 10, 20, 50]"
    showGridlines
    tableStyle="min-width: 50rem"
    tableClass="mt-3"
  >
    <Column field="province" header="Tỉnh/Thành"></Column>
    <Column field="name" header="Xã/Phường"></Column>
    <Column header="Gộp từ các phường">
      <template #body="slotProps">
        {{ slotProps.data?.mergedFrom?.join(", ") }}
      </template>
    </Column>
    <template #empty> Không có dữ liệu </template>
  </DataTable>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import DataTable from "primevue/datatable";
import Column from "primevue/column";
import InputText from "primevue/inputtext";
import Button from "primevue/button";

import responseApi from "./data/data.js";

const province = ref("hà nội");
const ward = ref("");
const datas = ref([]);
const tempDatas = ref([]);

onMounted(() => {
  datas.value = flattenedData(responseApi.data);
});

const onSearch = () => {
  if (province.value) {
    const provinceQuery = removeVietnameseTones(province.value);
    tempDatas.value = flattenedData(responseApi.data).filter((element) => {
      const itemName = removeVietnameseTones(element.province);
      return itemName.includes(provinceQuery);
    });
  } else {
    tempDatas.value = flattenedData(responseApi.data);
  }

  if (ward.value) {
    const wardQuery = removeVietnameseTones(ward.value);
    datas.value = tempDatas.value.filter((element) => {
      const itemName = removeVietnameseTones(element.name);
      return itemName.includes(wardQuery) || element.mergedFrom.some(e => {
        return removeVietnameseTones(e).includes(wardQuery)
      });
    });
  } else {
    datas.value = tempDatas.value;
  }
};

const removeVietnameseTones = (str) => {
  return str
    .toLowerCase()
    .normalize("NFD") // Chuyển thành chuỗi Unicode tổ hợp
    .replace(/[\u0300-\u036f]/g, "") // Loại bỏ các dấu
    .replace(/đ/g, "d") // Chuyển đ thành d
    .replace(/Đ/g, "D");
};

// Flatten wards
const flattenedData = (data) => {
  const demo = data.flatMap((item) =>
    item.wards.map((ward) => ({
      id: item.id,
      province: item.province,
      name: ward.name,
      mergedFrom: ward.mergedFrom,
    }))
  );
  
  return demo;
};
</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
