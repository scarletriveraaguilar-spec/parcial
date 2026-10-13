<script setup>
import { computed, onMounted, ref } from 'vue';
import axios from 'axios';
import { useI18n } from 'vue-i18n';
const { t } = useI18n();
const sections = ref([]);
const items = ref([]);
const loading = ref(true);
const error = ref('');
onMounted(async () => {
  try {
    const [s, i] = await Promise.all([
      axios.get('http://localhost:3000/showroom-sections'),
      axios.get('http://localhost:3000/vehicle-inventory-items')
    ]);
    sections.value = s.data; items.value = i.data;
  } catch (e) { error.value = t('inventory.apiError'); }
  finally { loading.value = false; }
});
const summaries = computed(() => sections.value.map(section => {
  const vehicles = items.value.filter(item => item.sectionId === section.id);
  const total = vehicles.reduce((sum, item) => sum + Number(item.quantity || 0), 0);
  return { ...section, vehicles, total, available: Math.max(0, section.maxCapacity - total) };
}));
</script>
<template>
  <section class="p-4 md:p-5">
    <div class="flex flex-wrap align-items-center justify-content-between gap-3 mb-5">
      <div><h1 class="text-4xl font-bold mb-2">{{ t('home.title') }}</h1><p class="m-0 text-color-secondary">{{ t('home.content') }}</p></div>
      <pv-button as="router-link" to="/vehicle/items/new" :label="t('inventory.newTitle')" icon="pi pi-plus" />
    </div>
    <h2 class="text-2xl font-semibold mb-3">{{ t('inventory.collections') }}</h2>
    <p v-if="loading">{{ t('inventory.loading') }}</p><p v-if="error" class="text-red-500">{{ error }}</p>
    <div class="grid">
      <div v-for="section in summaries" :key="section.id" class="col-12 md:col-6 xl:col-4">
        <pv-card>
          <template #title>{{ section.name }}</template>
          <template #content>
            <div v-if="section.vehicles.length" class="flex flex-column gap-2">
              <div v-for="vehicle in section.vehicles" :key="vehicle.id" class="flex justify-content-between gap-3">
                <span>{{ vehicle.vehicleName }}</span><strong>× {{ vehicle.quantity }}</strong>
              </div>
            </div>
            <p v-else class="text-color-secondary">{{ t('inventory.noVehicles') }}</p>
            <hr class="my-3" />
            <div class="flex justify-content-between"><span>{{ t('inventory.totalVehicles') }}</span><strong>{{ section.total }}</strong></div>
            <div class="flex justify-content-between mt-2"><span>{{ t('inventory.availableCapacity') }}</span><strong>{{ section.available }} / {{ section.maxCapacity }}</strong></div>
          </template>
        </pv-card>
      </div>
    </div>
  </section>
</template>
