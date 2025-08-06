<template>
  <div>
    <h2>Renovec - Exportação de Dados</h2>
    {{ store.birSelecionada }}

    <label>
      Mês (YYYY-MM):
      <input type="text" v-model="mes" placeholder="Ex: 2025-07" />
    </label>

    <label style="margin-left: 1rem;">
      BIR:
      <select v-model="bir">
        <option value="" disabled selected>Selecione o BIR</option> <!-- opção vazia padrão -->
        <option 
          v-for="birOption in birOptions" 
          :key="birOption" 
          :value="birOption.toString().padStart(8, '0')"
        >
          {{ birOption }}
        </option>
      </select>
    </label>

    <div style="margin-top: 1rem;">
      <button 
        @click="getExportWithPolling" 
        :disabled="!mes || !bir || carregando"
      >
        {{ carregando ? 'Carregando...' : 'getExport' }}
      </button>

      <button @click="limpar" style="margin-left: 1rem;">
        Limpar
      </button>

      <div v-if="carregando" class="loader" title="Carregando..."></div>
    </div>

    <div v-if="resultado" style="margin-top: 1.5rem;">
      <h3>Resultado da Exportação:</h3>
      <pre>{{ resultado }}</pre>

      <div v-if="resultado.links && resultado.links.length">
        <h4>Links disponíveis:</h4>
        <ul>
          <li v-for="(r, index) in resultado.links" :key="index">
            <a :href="r" target="_blank" rel="noopener noreferrer">{{ r }}</a>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useStore } from '@/stores/store';
import { watch } from 'vue'



const store = useStore();

const resultado = ref(null)
const carregando = ref(false)

store.birSelecionada = 0



function getMesAtualFormatado() {
  const hoje = new Date()
  const ano = hoje.getFullYear()
  const mes = (hoje.getMonth() + 1).toString().padStart(2, '0')
  return `${ano}-${mes}`
}

const mes = ref(getMesAtualFormatado()) 
// Inicializa bir com o primeiro da lista, já formatado com 8 dígitos (ex: 07600550)
const bir = ref('') // começa vazio, sem seleção

const birOptions = [7600382, 7600449, 7600550, 7600610, 7601112, 7601235, 7601236]

// Função para formatar o BIR com zeros à esquerda
function formatarBir(birNumber) {
  return birNumber.toString().padStart(8, '0')
}

let accessToken = ''
let requisicaoID = ''

function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

async function pegarTokenEBir(mesValue, birValue) {
  store.birSelecionada = birValue;
  try {
    carregando.value = true
    const response = await axios.post('https://api.mapsis.com.br/apiexpress/pegarequisicao', {
      mes: mesValue,
      bir: birValue
    })
    accessToken = response.data.token || ''
    requisicaoID = response.data.requestId || ''

    if (!accessToken || !requisicaoID) {
      throw new Error('Token ou requestId não retornados corretamente')
    }
  } catch (error) {
    carregando.value = false
    throw error
  }
  carregando.value = false
}

async function getExportWithPolling() {
  if (!mes.value || !bir.value) {
    alert('Preencha os campos Mês e BIR antes de continuar.')
    return
  }

  try {
    await pegarTokenEBir(mes.value, bir.value)

    carregando.value = true

    const configBase = {
      method: 'get',
      maxBodyLength: Infinity,
      headers: { 
        'client_id': '1cf5b30f-c8c6-4bd7-935e-53e327cf2763',
        'access_token': accessToken
      }
    }

    let tentativas = 0
    const maxTentativas = 10
    const delayMs = 3000

    while (tentativas < maxTentativas) {
      tentativas++

      const config = {
        ...configBase,
        url: `https://api-rci-qa-channels.bancorcibrasil.com.br/integration/renovaction-data-integration/v1/data-export/${requisicaoID}`
      }

      const response = await axios.request(config)
      const data = response.data

      console.log(`Tentativa ${tentativas}: status = ${data.status}`)

      if (data.status !== "PENDING" && data.links && data.links.length > 0) {
        resultado.value = data
        carregando.value = false
        return
      }

      await delay(delayMs)
    }

    resultado.value = { error: "Limite de tentativas atingido sem resposta final" }
    carregando.value = false

  } catch (error) {
    console.error('Erro ao obter exportação:', error)
    resultado.value = error.response?.data || error.message
    carregando.value = false
  }
}

function limpar() {
  mes.value = getMesAtualFormatado()
  bir.value = birOptions[2].toString().padStart(8, '0')
  resultado.value = null
  accessToken = ''
  requisicaoID = ''
}

watch(bir, (novoValor) => {
  store.birSelecionada = parseInt(novoValor)
})
</script>

<style scoped>
label {
  display: inline-block;
  margin-bottom: 0.5rem;
}
input, select {
  margin-left: 0.5rem;
  padding: 0.3rem;
  width: 150px;
}
button {
  padding: 0.5rem 1rem;
}

/* Loader estilo bolinha girando */
.loader {
  display: inline-block;
  margin-left: 1rem;
  width: 18px;
  height: 18px;
  border: 3px solid #ccc;
  border-top-color: #007bff; /* azul */
  border-radius: 50%;
  animation: spin 1s linear infinite;
  vertical-align: middle;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
