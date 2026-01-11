<script setup lang="ts">
import { ref } from 'vue';

// State
const mode = ref<'manual' | 'json'>('manual');
const query = ref('');
const referenceTexts = ref('');
const text1 = ref('');
const text2 = ref('');

// JSON Upload State
const file1 = ref<File | null>(null);
const file2 = ref<File | null>(null);
const errorMessage = ref('');

const handleFileUpload = async (event: Event, fileIndex: 1 | 2) => {
    errorMessage.value = '';
    const target = event.target as HTMLInputElement;
    if (target.files && target.files.length > 0) {
        const file = target.files[0];
        if (file) {
            if (fileIndex === 1) file1.value = file;
            else file2.value = file;
        }
    }

    // Auto-process if both files are present
    if (file1.value && file2.value) {
        await processFiles();
    }
};

const processFiles = async () => {
    try {
        const content1 = await readFile(file1.value!);
        const content2 = await readFile(file2.value!);

        const json1 = JSON.parse(content1);
        const json2 = JSON.parse(content2);

        validateJson(json1, 'File 1');
        validateJson(json2, 'File 2');

        if (json1.query !== json2.query) {
            throw new Error('query is not the same');
        }

        // Deep comparison for reference texts
        const ref1 = JSON.stringify(json1.references_texts);
        const ref2 = JSON.stringify(json2.references_texts);

        if (ref1 !== ref2) {
             throw new Error('references_texts is not the same');
        }

        query.value = json1.query;
        // Handle if reference_texts is an array or object by stringifying, otherwise just assign
        referenceTexts.value = typeof json1.references_texts === 'string'
            ? json1.references_texts
            : JSON.stringify(json1.references_texts, null, 2);

        text1.value = json1.raw_text;
        text2.value = json2.raw_text;

        // Switch to manual mode to show data
        mode.value = 'manual';
        errorMessage.value = ''; // Clear errors if successful

    } catch (e: any) {
        errorMessage.value = e.message;
    }
};

const validateJson = (json: any, fileName: string) => {
    const required = ['query', 'references_texts', 'raw_text'];
    for (const field of required) {
        if (!(field in json)) {
            throw new Error(`Field "${field}" missing in ${fileName}`);
        }
    }
};

const readFile = (file: File): Promise<string> => {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target?.result as string);
        reader.onerror = (e) => reject(e);
        reader.readAsText(file);
    });
};

const submitComparison = () => {
    // Submission logic here
    console.log('Submitting comparison:', { query: query.value, referenceTexts: referenceTexts.value, text1: text1.value, text2: text2.value });
};

</script>

<template>
  <div class="min-h-screen bg-gray-100 py-10 px-4 sm:px-6 lg:px-8 text-gray-800 font-sans">
    <div class="max-w-5xl mx-auto">
      <header class="mb-10 text-center">
        <h1 class="text-4xl font-extrabold text-gray-900 tracking-tight mb-2">Agent Response Comparator</h1>
        <p class="text-lg text-gray-600">Compare two AI generated responses against a query and reference.</p>
      </header>

      <div class="bg-white rounded-2xl shadow-xl border border-gray-200 overflow-hidden">
        <!-- Tabs -->
        <div class="flex border-b border-gray-200">
          <button
            @click="mode = 'manual'"
            class="flex-1 py-4 text-sm font-medium focus:outline-none transition-all duration-200"
            :class="mode === 'manual' ? 'bg-white text-indigo-600 border-b-2 border-indigo-600' : 'bg-gray-50 text-gray-500 hover:text-gray-700 hover:bg-gray-100'"
          >
            Data View / Manual Entry
          </button>
          <button
            @click="mode = 'json'"
            class="flex-1 py-4 text-sm font-medium focus:outline-none transition-all duration-200"
            :class="mode === 'json' ? 'bg-white text-indigo-600 border-b-2 border-indigo-600' : 'bg-gray-50 text-gray-500 hover:text-gray-700 hover:bg-gray-100'"
          >
            JSON Upload
          </button>
        </div>

        <div class="p-6 md:p-8">
            <transition name="fade" mode="out-in">
                <!-- JSON INPUT MODE -->
                <div v-if="mode === 'json'" key="json" class="max-w-xl mx-auto space-y-8 py-8">
                    <div class="text-center">
                        <h2 class="text-lg font-semibold text-gray-900">Upload Comparison Files</h2>
                        <p class="text-sm text-gray-500">Select two JSON files to compare. They must share the same query and references.</p>
                    </div>

                    <div class="space-y-6">
                        <div class="group relative">
                            <label class="block text-sm font-medium text-gray-700 mb-2">First JSON File</label>
                            <div class="mt-1 flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md hover:border-indigo-400 transition-colors">
                                <div class="space-y-1 text-center">
                                    <div class="flex text-sm text-gray-600 justify-center">
                                        <label for="file-upload-1" class="relative cursor-pointer bg-white rounded-md font-medium text-indigo-600 hover:text-indigo-500 focus-within:outline-none focus-within:ring-2 focus-within:ring-offset-2 focus-within:ring-indigo-500">
                                            <span>Upload a file</span>
                                            <input id="file-upload-1" name="file-upload-1" type="file" accept=".json" class="sr-only" @change="(e) => handleFileUpload(e, 1)">
                                        </label>
                                        <p class="pl-1">or drag and drop</p>
                                    </div>
                                    <p class="text-xs text-gray-500">JSON up to 10MB</p>
                                    <p v-if="file1" class="text-sm font-semibold text-green-600 mt-2">Selected: {{ file1.name }}</p>
                                </div>
                            </div>
                        </div>

                        <div class="group relative">
                            <label class="block text-sm font-medium text-gray-700 mb-2">Second JSON File</label>
                            <div class="mt-1 flex justify-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md hover:border-indigo-400 transition-colors">
                                <div class="space-y-1 text-center">
                                    <div class="flex text-sm text-gray-600 justify-center">
                                        <label for="file-upload-2" class="relative cursor-pointer bg-white rounded-md font-medium text-indigo-600 hover:text-indigo-500 focus-within:outline-none focus-within:ring-2 focus-within:ring-offset-2 focus-within:ring-indigo-500">
                                            <span>Upload a file</span>
                                            <input id="file-upload-2" name="file-upload-2" type="file" accept=".json" class="sr-only" @change="(e) => handleFileUpload(e, 2)">
                                        </label>
                                        <p class="pl-1">or drag and drop</p>
                                    </div>
                                    <p class="text-xs text-gray-500">JSON up to 10MB</p>
                                    <p v-if="file2" class="text-sm font-semibold text-green-600 mt-2">Selected: {{ file2.name }}</p>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div v-if="errorMessage" class="p-4 bg-red-50 border border-red-200 text-red-700 rounded-md text-sm flex items-start">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 mt-0.5 flex-shrink-0" viewBox="0 0 20 20" fill="currentColor">
                            <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
                        </svg>
                        <span>{{ errorMessage }}</span>
                    </div>
                </div>

                <!-- MANUAL INPUT MODE -->
                <div v-else key="manual" class="space-y-8">
                    <div class="grid grid-cols-1 gap-6">
                        <div>
                            <label class="block text-sm font-semibold text-gray-700 mb-2">Query</label>
                            <textarea
                                v-model="query"
                                rows="3"
                                class="w-full rounded-lg border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition-shadow"
                                placeholder="Enter the user query..."
                            ></textarea>
                        </div>

                        <div>
                            <label class="block text-sm font-semibold text-gray-700 mb-2">Reference Texts</label>
                            <textarea
                                v-model="referenceTexts"
                                rows="4"
                                class="w-full rounded-lg border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition-shadow"
                                placeholder="Enter reference ground truth texts..."
                            ></textarea>
                        </div>
                    </div>

                    <div class="border-t border-gray-100 pt-8">
                         <h3 class="text-lg font-medium text-gray-900 mb-4">Responses to Compare</h3>
                         <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                            <div class="bg-gray-50 p-4 rounded-lg border border-gray-200">
                                <label class="block text-sm font-semibold text-gray-700 mb-2">Raw Text (Response A)</label>
                                <textarea
                                    v-model="text1"
                                    rows="12"
                                    class="w-full rounded-md border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 text-gray-600 leading-relaxed"
                                    placeholder="Paste response A here..."
                                ></textarea>
                            </div>
                            <div class="bg-gray-50 p-4 rounded-lg border border-gray-200">
                                <label class="block text-sm font-semibold text-gray-700 mb-2">Raw Text (Response B)</label>
                                <textarea
                                    v-model="text2"
                                    rows="12"
                                    class="w-full rounded-md border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 text-gray-600 leading-relaxed"
                                    placeholder="Paste response B here..."
                                ></textarea>
                            </div>
                        </div>
                    </div>

                    <div class="flex justify-center pt-6">
                        <button
                            @click="submitComparison"
                            class="inline-flex items-center px-6 py-3 border border-transparent text-base font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-colors"
                        >
                            Submit Comparison
                        </button>
                    </div>
                </div>
            </transition>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

