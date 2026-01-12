<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps<{
  data: any;
}>();

// Define dimensions to display order (excluding overall)
const dimensions = [
  { key: 'correctness_topical', label: 'Topical Correctness' },
  { key: 'coherence_logical', label: 'Logical Coherence' },
  { key: 'coherence_stylistic', label: 'Stylistic Coherence' },
  { key: 'coverage_broad', label: 'Broad Coverage' },
  { key: 'coverage_deep', label: 'Deep Coverage' },
  { key: 'consistency_internal', label: 'Internal Consistency' },
];

const getWinnerClass = (winner: string) => {
    const w = winner ? winner.toLowerCase() : '';
    if (w === 'a') return 'bg-indigo-100 text-indigo-800 border-indigo-200';
    if (w === 'b') return 'bg-purple-100 text-purple-800 border-purple-200';
    return 'bg-gray-100 text-gray-800 border-gray-200'; // Tie or n
};

const getWinnerText = (winner: string) => {
    const w = winner ? winner.toLowerCase() : '';
    if (w === 'a') return 'Winner: Response A';
    if (w === 'b') return 'Winner: Response B';
    if (w === 'n') return 'Tie because the score is to close!';
    return 'Error';
};

const overall = computed(() => {
    return props.data.quality_overall || {};
});

const getOverallWinner = computed(() => {
    const winner = overall.value.agent_winner;
    if (winner === 'a') return 'Response A';
    if (winner === 'b') return 'Response B';
    return 'Tie';
});
</script>

<template>
  <div class="space-y-8 animate-fade-in-up">
    <!-- Overall Summary -->
    <div class="bg-gradient-to-r from-gray-900 via-indigo-900 to-purple-900 rounded-xl shadow-lg p-8 text-white">
        <div class="flex flex-col md:flex-row justify-between items-center">
            <div class="mb-6 md:mb-0">
                <h2 class="text-3xl font-extrabold tracking-tight mb-2">Final Verdict</h2>
                <div class="text-xl opacity-90">
                    Winner: <span class="font-bold text-yellow-300">{{ getOverallWinner }}</span>
                </div>
            </div>
            
            <div class="flex space-x-8 text-center">
                <div class="bg-white/10 rounded-lg p-4 backdrop-blur-sm min-w-[120px]">
                    <div class="text-xs uppercase tracking-widest opacity-75 mb-1">Score A</div>
                    <div class="text-3xl font-bold">{{ overall.score_a != null ? overall.score_a : '-' }}</div>
                </div>
                <div class="bg-white/10 rounded-lg p-4 backdrop-blur-sm min-w-[120px]">
                    <div class="text-xs uppercase tracking-widest opacity-75 mb-1">Score B</div>
                    <div class="text-3xl font-bold">{{ overall.score_b != null ? overall.score_b : '-' }}</div>
                </div>
            </div>
        </div>
    </div>

    <!-- Comparative Breakdown -->
    <div>
        <h3 class="text-2xl font-bold text-gray-900 mb-6">Detailed Comparison</h3>
        <div class="grid grid-cols-1 gap-6">
            <div 
                v-for="dim in dimensions" 
                :key="dim.key" 
                class="bg-white border rounded-lg overflow-hidden shadow-sm hover:shadow-md transition-shadow"
            >
                <div class="flex flex-col md:flex-row md:items-stretch">
                   <!-- Dimension Header & Winner -->
                   <div 
                       class="md:w-1/4 p-5 flex flex-col justify-center border-b md:border-b-0 md:border-r"
                       :class="data[dim.key] ? 'border-gray-100' : 'border-gray-100 opacity-50'"
                   >
                        <h4 class="text-lg font-bold text-gray-800 mb-3">{{ dim.label }}</h4>
                        <div v-if="data[dim.key]">
                            <span 
                                class="inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold border"
                                :class="getWinnerClass(data[dim.key].agent_winner)"
                            >
                                {{ getWinnerText(data[dim.key].agent_winner) }}
                            </span>
                        </div>
                   </div>

                   <!-- Explanation -->
                   <div class="md:w-3/4 p-5 bg-gray-50 flex items-center">
                        <p v-if="data[dim.key]" class="text-gray-700 leading-relaxed text-sm">
                            {{ data[dim.key].explantion || data[dim.key].explanation || 'No explanation provided.' }}
                        </p>
                        <p v-else class="text-gray-400 italic text-sm">No comparison data available.</p>
                   </div>
                </div>
            </div>
        </div>
    </div>
  </div>
</template>

<style scoped>
.animate-fade-in-up {
  animation: fadeInUp 0.5s ease-out;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
