<template>
  <input type="file" @change="handleFile" accept=".csv" />
  <p v-if="csvFile">Arquivo: {{ csvFile.name }}</p>
  <button @click="uploadFile" :disabled="!csvFile">Enviar CSV</button>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useStore } from '@/stores/store';

const store = useStore();

const csvFile = ref(null)

function handleFile(event) {
  csvFile.value = event.target.files[0]
}

async function uploadFile() {
  const formData = new FormData()
  formData.append('file', csvFile.value)

  // Incluindo a coluna BIR com valor da store
  formData.append('BIR', store.birSelecionada)

  try {
    const response = await axios.post('https://api.mapsis.com.br/apiexpress/inserirRenaultCSV',  formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    })
    alert('Importado com sucesso!')
  } catch (err) {
    console.error(err)
    alert('Erro ao importar CSV')
  }
}
</script>
