<script setup lang="ts">
import { ref } from 'vue';

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

// Accordion State
const activeDimension = ref<string | null>(null);

const toggleDimension = (key: string) => {
  if (activeDimension.value === key) {
    activeDimension.value = null; // Close if already open
  } else {
    activeDimension.value = key; // Open this one
  }
};
</script>

<template>
  <div class="space-y-4">
    <div v-for="dim in dimensions" :key="dim.key" class="border rounded-lg bg-white shadow-sm overflow-hidden">
      <!-- Accordion Header -->
      <button 
        @click="toggleDimension(dim.key)"
        class="w-full flex justify-between items-center p-5 bg-white hover:bg-gray-50 transition-colors focus:outline-none"
      >
        <div class="flex items-center">
            <h3 class="text-lg font-bold text-gray-800 capitalize">{{ dim.label }}</h3>
            <!-- Optional: Status indicator if data exists? -->
            <span v-if="!data[dim.key]" class="ml-3 text-xs text-gray-400 font-normal italic">(No Data)</span>
        </div>
        
        <svg 
            xmlns="http://www.w3.org/2000/svg" 
            class="h-5 w-5 text-gray-400 transform transition-transform duration-200"
            :class="activeDimension === dim.key ? 'rotate-180' : ''"
            viewBox="0 0 20 20" 
            fill="currentColor"
        >
            <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
        </svg>
      </button>
      
      <!-- Accordion Content -->
      <transition
        enter-active-class="transition duration-100 ease-out"
        enter-from-class="transform scale-95 opacity-0"
        enter-to-class="transform scale-100 opacity-100"
        leave-active-class="transition duration-75 ease-in"
        leave-from-class="transform scale-100 opacity-100"
        leave-to-class="transform scale-95 opacity-0"
      >
        <div v-show="activeDimension === dim.key" class="border-t border-gray-100 p-5 bg-gray-50/50">
          <div v-if="data[dim.key]" class="space-y-4">
            <!-- Overall Judgement -->
            <div class="bg-blue-50/50 p-4 rounded-md border-l-4 border-indigo-500">
                <h4 class="text-xs font-bold text-indigo-500 uppercase tracking-wider mb-1">Overall Judgement</h4>
                <p class="text-gray-800 leading-relaxed text-sm">{{ data[dim.key].overall_judgement }}</p>
            </div>

            <!-- Judgement Details -->
            <div v-if="data[dim.key].judgement_details && data[dim.key].judgement_details.length > 0">
                <h4 class="text-sm font-bold text-gray-700 mt-4 mb-3">Key Issues & Points</h4>
                <div class="space-y-3">
                    <div v-for="(detail, idx) in data[dim.key].judgement_details" :key="idx" class="bg-white border border-gray-100 rounded-md p-3 shadow-sm">
                        <div class="flex items-center mb-2">
                            <span class="px-2 py-0.5 bg-red-100 text-red-800 text-xs font-bold rounded uppercase tracking-wide">{{ formatType(detail.type) }}</span>
                        </div>
                        
                        <div class="space-y-2">
                            <div v-if="detail.statement" class="text-sm border-l-2 border-gray-300 pl-3 italic text-gray-500 my-1">
                                "{{ detail.statement }}"
                            </div>
                            <p class="text-sm text-gray-800"><span class="font-semibold text-gray-600">Explanation:</span> {{ detail.explanation }}</p>
                        </div>
                    </div>
                </div>
            </div>
            <div v-else class="text-sm text-gray-500 italic px-2">No specific issues flagged details for this dimension.</div>
          </div>
          <div v-else class="text-gray-400 italic text-sm p-2">No data returned for this dimension.</div>
        </div>
      </transition>
    </div>
  </div>
</template>
