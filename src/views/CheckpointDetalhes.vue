<template>
    <div class="max-w-3xl mx-auto p-4 bg-white rounded-lg shadow-md">
      <button
        class="mb-6 text-blue-600 hover:text-blue-800 font-semibold transition-colors"
        @click="$emit('voltar')"
      >
        ← Voltar
      </button>
  
      <h2 class="text-2xl font-bold mb-4 text-gray-700">Detalhes: {{ titulo }}</h2>
  
 

      <div
        v-for="(item, index) in itens"
        :key="index"
        class="mb-6 pb-4 border-b-2 border-gray-300 last:border-b-0"
      >
        <p><strong>Descrição Tipo:</strong> {{ item.DESC_TIPO }}</p>  
        <pre class="json-code">{{ item.DESCRICAO }}</pre>
      </div>

    </div>
  </template>
  
  <script setup>
  defineProps({
    titulo: String,
    itens: Array
  })
  defineEmits(['voltar'])
  </script>
  
  <style scoped>
  .json-code {
    background-color: #2d2d2d;
    color: #f8f8f2;
    padding: 1rem;
    border-radius: 6px;
    font-family: 'Fira Code', monospace, monospace;
    font-size: 0.9rem;
    white-space: pre-wrap; /* permite quebra de linha */
    overflow-x: auto;
    box-shadow: inset 0 0 5px #000;
    line-height: 1.4;
  }
  </style>
  