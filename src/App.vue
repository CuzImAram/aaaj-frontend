<script setup lang="ts">
import { ref } from 'vue';

// State
const mode = ref<'manual' | 'json'>('manual');
const query = ref('');
const referenceTexts = ref<string[]>(['', '', '']); // Default 3 empty references
const isEditMode = ref(false);
const rawReferenceText = ref('');
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

const removeFile = (fileIndex: 1 | 2) => {
    if (fileIndex === 1) {
        file1.value = null;
        // Reset input value to allow re-selecting the same file
        const input = document.getElementById('file-upload-1') as HTMLInputElement;
        if (input) input.value = '';
    } else {
        file2.value = null;
        const input = document.getElementById('file-upload-2') as HTMLInputElement;
        if (input) input.value = '';
    }
    errorMessage.value = '';
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
             throw new Error('references_texts are not the same');
        }

        query.value = json1.query;
        
        // Assign directly as it's validated to be a list of strings
        referenceTexts.value = json1.references_texts;
        // Ensure we have at least 3 or match the length
        if (referenceTexts.value.length === 0) referenceTexts.value = ['', '', ''];

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

    // Validate references_texts is a List of Strings
    if (!Array.isArray(json.references_texts)) {
        throw new Error(`references_texts must be a List in ${fileName}`);
    }
    if (!json.references_texts.every((item: any) => typeof item === 'string')) {
        throw new Error(`references_texts must contain only strings in ${fileName}`);
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

const toggleEditMode = () => {
    if (isEditMode.value) {
        // Switching from Edit Mode to List View -> Parse
        try {
            const content = rawReferenceText.value.trim();
            if (content) {
                 // Wrap in brackets to parse as JSON array: "A",\n"B" -> ["A", "B"]
                 const parsed = JSON.parse(`[${content}]`);
                 if (Array.isArray(parsed) && parsed.every(i => typeof i === 'string')) {
                     referenceTexts.value = parsed;
                     isEditMode.value = false;
                     errorMessage.value = '';
                 } else {
                     throw new Error('Invalid format');
                 }
            } else {
                referenceTexts.value = ['', '', ''];
                isEditMode.value = false;
            }
        } catch (e) {
            errorMessage.value = 'Invalid reference text format. Expected: "Ref 1",\n"Ref 2"';
        }
    } else {
        // Switching from List View to Edit Mode -> Format
        rawReferenceText.value = referenceTexts.value
            .map(r => `"${r.replace(/"/g, '\\"')}"`) // Escape quotes
            .join(',\n');
        isEditMode.value = true;
    }
};

const addReference = () => {
    referenceTexts.value.push('');
};

const removeReference = (index: number) => {
    referenceTexts.value.splice(index, 1);
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
        <p class="text-lg text-gray-600">Compare two responses against a query and reference.</p>
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
                                    <div v-if="file1" class="mt-2 flex items-center justify-center space-x-2">
                                        <span class="text-sm font-semibold text-green-600">Selected: {{ file1.name }}</span>
                                        <button @click="removeFile(1)" class="text-red-500 hover:text-red-700 focus:outline-none">
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                                <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                                            </svg>
                                        </button>
                                    </div>
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
                                    <div v-if="file2" class="mt-2 flex items-center justify-center space-x-2">
                                        <span class="text-sm font-semibold text-green-600">Selected: {{ file2.name }}</span>
                                        <button @click="removeFile(2)" class="text-red-500 hover:text-red-700 focus:outline-none">
                                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                                <path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" />
                                            </svg>
                                        </button>
                                    </div>
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
                            <div class="flex justify-between items-center mb-2">
                                <label class="block text-sm font-semibold text-gray-700">Reference Texts</label>
                                <button 
                                    @click="toggleEditMode" 
                                    class="text-xs font-medium text-indigo-600 hover:text-indigo-800 focus:outline-none"
                                >
                                    {{ isEditMode ? 'Switch to List View' : 'Edit Raw Text' }}
                                </button>
                            </div>

                            <!-- List View -->
                            <div v-if="!isEditMode" class="space-y-3">
                                <div v-for="(text, index) in referenceTexts" :key="index" class="flex items-start space-x-3">
                                    <span class="mt-2 text-xs font-mono text-gray-400 w-4 text-right">{{ index + 1 }}.</span>
                                    <textarea
                                        v-model="referenceTexts[index]"
                                        rows="2"
                                        class="flex-1 rounded-lg border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition-shadow"
                                        placeholder="Enter reference text..."
                                    ></textarea>
                                    <button 
                                        @click="removeReference(index)" 
                                        class="mt-2 text-gray-400 hover:text-red-500 focus:outline-none transition-colors"
                                        title="Remove reference"
                                    >
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" />
                                        </svg>
                                    </button>
                                </div>
                                <button 
                                    @click="addReference" 
                                    class="mt-2 inline-flex items-center text-sm font-medium text-indigo-600 hover:text-indigo-800 focus:outline-none"
                                >
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
                                        <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
                                    </svg>
                                    Add Reference
                                </button>
                            </div>

                            <!-- Edit Mode -->
                            <div v-else>
                                <textarea
                                    v-model="rawReferenceText"
                                    rows="6"
                                    class="w-full rounded-lg border-gray-300 border shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm p-3 transition-shadow font-mono"
                                    placeholder='"Reference 1",&#10;"Reference 2"'
                                ></textarea>
                                <p class="mt-2 text-sm text-gray-500">
                                    <strong>Format:</strong> Each reference in double quotes <code>"..."</code>, separated by a comma <code>,</code> and a new line.
                                </p>
                            </div>
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

