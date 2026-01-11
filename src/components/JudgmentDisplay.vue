<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  data: any;
}>();

// Define dimensions to display order
const dimensions = [
  { key: 'correctness_topical', label: 'Topical Correctness' },
  { key: 'coherence_logical', label: 'Logical Coherence' },
  { key: 'coherence_stylistic', label: 'Stylistic Coherence' },
  { key: 'coverage_broad', label: 'Broad Coverage' },
  { key: 'coverage_deep', label: 'Deep Coverage' },
  { key: 'consistency_internal', label: 'Internal Consistency' },
];

const formatType = (type: string) => {
  return type;
};
</script>

<template>
  <div class="space-y-8">
    <div v-for="dim in dimensions" :key="dim.key" class="border rounded-lg p-5 bg-white shadow-sm">
      <h3 class="text-xl font-bold text-gray-800 mb-3 capitalize">{{ dim.label }}</h3>
      
      <div v-if="data[dim.key]" class="space-y-4">
        <!-- Overall Judgement -->
        <div class="bg-gray-50 p-4 rounded-md border-l-4 border-indigo-500">
            <h4 class="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-1">Overall Judgement</h4>
            <p class="text-gray-800 leading-relaxed">{{ data[dim.key].overall_judgement }}</p>
        </div>

        <!-- Judgement Details -->
        <div v-if="data[dim.key].judgement_details && data[dim.key].judgement_details.length > 0">
            <h4 class="text-sm font-bold text-gray-700 mt-4 mb-2">Details</h4>
            <div class="space-y-3">
                <div v-for="(detail, idx) in data[dim.key].judgement_details" :key="idx" class="border-t border-gray-100 pt-3 pb-1">
                    <div class="flex items-center mb-2">
                        <span class="px-2 py-1 bg-red-100 text-red-800 text-xs font-semibold rounded-full">{{ formatType(detail.type) }}</span>
                    </div>
                    
                    <div class="space-y-2 pl-1">
                        <div v-if="detail.statement" class="text-sm border-l-2 border-gray-300 pl-3 italic text-gray-600 bg-gray-50 py-1 rounded-r">
                            "{{ detail.statement }}"
                        </div>
                        <p class="text-sm text-gray-800"><span class="font-semibold">Explanation:</span> {{ detail.explanation }}</p>
                    </div>
                </div>
            </div>
        </div>
        <div v-else class="text-sm text-gray-500 italic">No specific issues found.</div>
      </div>
      <div v-else class="text-gray-500 italic">No data returned for this dimension.</div>
    </div>
  </div>
</template>
