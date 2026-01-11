<script setup lang="ts">
import { ref, computed, nextTick, watch } from 'vue';

const props = defineProps<{
  label: string;
  modelValue: string;
  references: string[];
  placeholder?: string;
}>();

const emit = defineEmits(['update:modelValue']);

const isEditing = ref(false);
const textareaRef = ref<HTMLTextAreaElement | null>(null);

// Focus textarea when entering edit mode
watch(isEditing, async (newVal) => {
  if (newVal) {
    await nextTick();
    textareaRef.value?.focus();
  }
});

const renderedText = computed(() => {
  if (!props.modelValue) return '';

  // Escape HTML to prevent XSS (basic)
  let text = props.modelValue
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");

  // Replace [n] with highlighted spans
  // Regex looks for [digits]
  return text.replace(/\[(\d+)\]/g, (match, number) => {
    const index = parseInt(number, 10) - 1; // 1-based to 0-based
    const isValid = index >= 0 && index < props.references.length;
    const referenceText = isValid ? props.references[index] : 'Reference not found';
    
    // Show full reference text as requested
    const tooltipText = referenceText || '';

    const colorClass = isValid ? 'text-blue-600 font-bold cursor-help' : 'text-red-600 font-bold cursor-help';
    
    // Tooltip positioning: "bottom corner of the right ]"
    // We align it to the right (right-0) and put it below (top-full) with a small margin (mt-1)
    // Use 'inline' instead of 'inline-block' to prevent breaking text flow
    // Use 'inline' instead of 'inline-block' to prevent breaking text flow
    // IMPORTANT: No newlines in the template string because whitespace-pre-wrap will render them!
    return `<span class="group relative inline ${colorClass}">${match}<span class="pointer-events-none absolute top-full right-0 mt-1 w-80 p-3 bg-gray-900 text-white text-xs rounded-md shadow-xl opacity-0 group-hover:opacity-100 transition-opacity z-20 whitespace-normal break-words text-left">${tooltipText}<!-- Arrow pointing up to the right side --><svg class="absolute text-gray-900 h-2 w-4 right-2 bottom-full transform rotate-180" x="0px" y="0px" viewBox="0 0 255 255" xml:space="preserve"><polygon class="fill-current" points="0,0 127.5,127.5 255,0"/></svg></span></span>`;
  });
});

const updateValue = (event: Event) => {
  const target = event.target as HTMLTextAreaElement;
  emit('update:modelValue', target.value);
};

const enableEdit = () => {
  isEditing.value = true;
};

const disableEdit = () => {
  isEditing.value = false;
};
</script>

<template>
  <div class="bg-gray-50 p-4 rounded-lg border border-gray-200 flex flex-col h-full">
    <div class="flex justify-between items-center mb-2">
        <label class="block text-sm font-semibold text-gray-700">{{ label }}</label>
        <span class="text-xs text-gray-500">
            Use <code>[n]</code> to cite reference #n.
        </span>
    </div>
    
    <div class="relative flex-grow min-h-[300px]">
        <!-- View Mode -->
        <div 
            v-if="!isEditing"
            @click="enableEdit"
            class="w-full h-full rounded-md border border-gray-300 bg-white p-3 text-gray-600 leading-relaxed whitespace-pre-wrap cursor-text hover:border-indigo-300 transition-colors overflow-y-auto max-h-[500px]"
            v-html="renderedText || '<span class=\'text-gray-400\'>' + (placeholder || 'Enter text here...') + '</span>'"
        ></div>

        <!-- Edit Mode -->
        <textarea
            v-else
            ref="textareaRef"
            :value="modelValue"
            @input="updateValue"
            @blur="disableEdit"
            rows="12"
            class="w-full h-full rounded-md border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 text-gray-600 leading-relaxed"
            :placeholder="placeholder"
        ></textarea>
    </div>
  </div>
</template>
