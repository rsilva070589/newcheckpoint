<template>
  <div class="max-w-3xl mx-auto p-6 bg-white rounded-2xl shadow-lg border border-gray-200">
    <!-- Botão Voltar -->
    <button
      class="mb-6 flex items-center gap-2 text-blue-600 hover:text-blue-800 font-semibold transition-colors"
      @click="$emit('voltar')"
    >
      <span class="text-xl">←</span> Voltar
    </button>

    <!-- Título -->
    <h2 class="text-3xl font-bold mb-6 text-gray-800 border-b pb-2">
      Detalhes: {{ titulo }}
    </h2>

    <!-- Lista -->
    <div
      v-for="(item, index) in itens"
      :key="index"
      class="mb-8 pb-6 border-b border-gray-300 last:border-b-0"
    >
      <p class="mb-2 text-gray-700">
        <strong class="text-gray-900">Descrição Tipo:</strong> {{ item.DESC_TIPO }}
      </p>

      <pre class="json-code mb-4">{{ item.DESCRICAO }}</pre>
 
      <button v-if="item.COD_TIPO == 5"      
        class="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg shadow-md transition-all duration-200"
   @click="handleReenvio(item.id_registro, 5, index)"
      >
        Reenviar
      </button>

      <!-- Mensagem de sucesso -->
      <p v-if="mensagemSucesso[index]" class="mt-2 text-green-600 font-semibold">
        ✅ Reenviado com sucesso!
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

defineProps({
  titulo: String,
  itens: Array
})
defineEmits(['voltar'])

// Array para controlar mensagens de sucesso por item
const mensagemSucesso = ref([])

async function handleReenvio(opportunityId, tipo, index) {
  try {
    await axios.post('https://api.mapsis.com.br/apiexpress/reenvia', {
      opportunityId,
      tipo
    });

    // Marca sucesso para este item
    mensagemSucesso.value[index] = true;

    // Limpa a mensagem depois de 3 segundos
    setTimeout(() => {
      mensagemSucesso.value[index] = false;
    }, 3000);

  } catch (err) {
    console.error('❌ Erro ao enviar requisição:', err.response?.data || err.message);
    alert('Erro ao reenviar!'); // opcional
  }
}

</script>


<style scoped>
.json-code {
  background-color: #1e1e2f;
  color: #e4e4e7;
  padding: 1rem;
  border-radius: 8px;
  font-family: 'Fira Code', monospace;
  font-size: 0.9rem;
  white-space: pre-wrap; /* permite quebra de linha */
  overflow-x: auto;
  box-shadow: inset 0 0 6px rgba(0,0,0,0.4);
  line-height: 1.5;
}
</style>
