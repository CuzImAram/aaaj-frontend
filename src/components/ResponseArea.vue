<script setup lang="ts">
import { ref, computed, nextTick, watch } from 'vue';

const props = defineProps<{
  label: string;
  modelValue: string;
  references: string[];
  placeholder?: string;
}>();

const emit = defineEmits(['update:modelValue', 'hover-reference', 'leave-reference', 'invalid-reference']);

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

  // Regex looks for brackets containing digits, spaces, and commas
  return text.replace(/\[([\d\s,]+)\]/g, (fullMatch, innerContent) => {
    // Check validity of all numbers in this block
    const numbers = innerContent.match(/\d+/g) || [];
    const indices = numbers.map((n: string) => parseInt(n, 10));
    const allValid = indices.length > 0 && indices.every((i: number) => i >= 0 && i < props.references.length);
    const anyInvalid = indices.some((i: number) => i < 0 || i >= props.references.length);

    // Container style: Blue if all valid, Red if all invalid? Or just bold default.
    // User asked for "everything blue" for valid case.
    let containerClass = 'font-bold';
    if (allValid) {
        containerClass += ' text-blue-600';
    } else if (indices.length > 0 && indices.every((i: number) => i < 0 || i >= props.references.length)) {
         // Optional: if ALL are invalid, make brackets red too?
         containerClass += ' text-red-600';
    }

    // Process the inner content
    const processedInner = innerContent.replace(/(\d+)/g, (digitMatch: string) => {
        const index = parseInt(digitMatch, 10);
        const isValid = index >= 0 && index < props.references.length;
        
        let itemClass = 'cursor-pointer hover:underline';
        
        // If container is NOT blue (mixed or all invalid), we must color valid items blue and invalid items red explicitly.
        // If container IS blue (all valid), items inherit blue.
        if (!allValid) {
             itemClass += isValid ? ' text-blue-600' : ' text-red-600';
        }
        
        return `<span class="ref-item ${itemClass}" data-index="${index}">${digitMatch}</span>`;
    });

    return `<span class="${containerClass}">[${processedInner}]</span>`;
  });
});

const handleClick = (event: MouseEvent) => {
    const target = event.target as HTMLElement;
    if (target.classList.contains('ref-item')) {
        // Prevent edit mode when clicking a citation
        event.stopPropagation();
        
        const index = parseInt(target.dataset.index || '-1', 10);
        if (index >= 0 && index < props.references.length) {
            const el = document.getElementById(`ref-${index}`);
            if (el) {
                el.scrollIntoView({ behavior: 'smooth', block: 'center' });
                
                el.scrollIntoView({ behavior: 'smooth', block: 'center' });
                
                const textarea = el.querySelector('textarea');
                if (textarea) {
                    // Highlight ONLY the textarea
                    const highlightClasses = ['bg-blue-50', 'ring-4', 'ring-blue-300', 'transition-colors', 'duration-500'];
                    textarea.classList.add(...highlightClasses);

                    setTimeout(() => {
                        textarea.classList.remove(...highlightClasses);
                    }, 2000);
                }
            }
        } else {
            // Invalid reference
            emit('invalid-reference', index);
        }
    } else {
        // Normal click => Edit mode
        enableEdit();
    }
};

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
            Use <code>[n]</code> or <code>[x, y]</code> to cite reference #n.
        </span>
    </div>
    
    <div class="relative flex-grow min-h-[300px]">
        <!-- View Mode -->
        <div 
            v-if="!isEditing"
            @click="handleClick"
            class="w-full h-full rounded-md border border-gray-300 bg-white p-3 text-gray-600 leading-relaxed whitespace-pre-wrap break-words overflow-x-hidden cursor-text hover:border-indigo-300 transition-colors overflow-y-auto max-h-[500px]"
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
