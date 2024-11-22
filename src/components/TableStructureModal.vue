<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg shadow-xl w-11/12 max-w-4xl h-4/5 overflow-hidden">
      <div class="p-4 border-b border-gray-200 flex justify-between items-center">
        <h3 class="text-lg font-semibold text-gray-900">表结构分析</h3>
        <button @click="$emit('close')" class="text-gray-500 hover:text-gray-700">
          <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
      <div class="p-6 overflow-auto h-[calc(100%-5rem)]">
        <div v-if="structure.length === 0" class="text-center text-gray-500">
          未检测到表结构信息
        </div>
        <div v-else class="space-y-4">
          <div v-for="table in structure" :key="table.name" class="bg-gray-50 rounded-lg p-4">
            <div class="flex justify-between items-center mb-3">
              <div class="font-semibold text-gray-900">{{ table.name }}</div>
              <div class="text-sm text-gray-500">别名: [{{ table.alias }}]</div>
            </div>
            <div class="pl-4 grid grid-cols-3 gap-2">
              <div v-for="field in table.fields" :key="field.name" class="text-sm text-gray-700 bg-white rounded p-2">
                {{ field.name }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  structure: {
    type: Array,
    required: true,
    default: () => []
  }
})

defineEmits(['close'])
</script>

<style scoped>
.h-4\/5 {
  height: 80vh;
}
</style>
