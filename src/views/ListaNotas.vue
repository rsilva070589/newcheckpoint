<template>
 
 
  <div class="p-8 bg-gray-50 min-h-screen" style="margin-left: -30px;">
  
    <div class="flex gap-2 mb-6">
  <input
    v-model="notaBusca"
    type="text"
    placeholder="Buscar ID_Document Expecifico..."
    class="p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
  />
  <button
    @click="buscarNota"
    class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded shadow transition"
  >
    Buscar ID_Document
  </button>
</div>
    <!-- Filtros -->
    <div class="flex flex-wrap gap-4 max-w-5xl mb-8">
      <select style="height: 35px;"
        v-model="filtroStatus"
        class="flex-grow min-w-[220px] p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      >
        <option value="">Todos os Status</option>
        <option value="1">Sucesso</option>
        <option value="0">Pendente</option>
        <option value="2">Erro</option>
      </select>
      
      <input
        v-model="filtroGeral"
        type="text"
        placeholder="Filtro geral..."
        class="flex-grow min-w-[220px] p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      />
      <input
        v-model="filtroFornecedor"
        type="text"
        placeholder="Filtrar por Fornecedor..."
        class="flex-grow min-w-[220px] p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      />
      <select style="height: 35px;"
        v-model="filtroEmpresa"
        class="flex-grow min-w-[220px] p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      >
        <option value="">Filtrar por Código Empresa...</option>
        <option
          v-for="empresa in empresas"
          :key="empresa.codigo + empresa.nome"
          :value="empresa.codigo"
        >
          {{ empresa.nome }} ({{ empresa.codigo }})
        </option>
      </select>
      <input
        v-model="filtroDataInicio"
        type="date"
        class="p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      />
      <input
        v-model="filtroDataFim"
        type="date"
        class="p-3 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 transition"
      />
    </div>

    <!-- Status -->
    <div v-if="loading" class="text-gray-500 italic mb-4">Carregando notas...</div>
    <div v-else-if="error" class="text-red-600 font-semibold mb-4">Erro ao carregar: {{ error }}</div>

    <!-- Tabela -->
    <div v-else class="overflow-x-auto shadow-lg rounded-lg bg-white">
      <table class="w-full table-auto border-collapse">
        <thead class="bg-indigo-100">
          <tr>
            <th class="border px-4 py-2 text-center text-indigo-700">Ações</th>
            <th class="border px-4 py-2 text-indigo-700">Fornecedor</th>
            <th class="border px-4 py-2 text-indigo-700">Data</th>
      
            <th class="border px-4 py-2 text-indigo-700">Nota</th>
            <th class="border px-4 py-2 text-indigo-700">Empresa</th>
            <th class="border px-4 py-2 text-indigo-700">Cliente</th>
            <th class="border px-4 py-2 text-indigo-700">Data Nota</th>
            <th class="border px-4 py-2 text-indigo-700">Valor</th>
            <th class="border px-4 py-2 text-indigo-700">Natureza</th>
            <th class="border px-4 py-2 text-indigo-700">Comentário</th>
            <th class="border px-4 py-2 text-indigo-700">STATUS</th>
            <th class="border px-4 py-2 text-indigo-700">RESULTADO</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="nota in notasPaginadas"
            :key="nota.ID_DOCUMENT"
            :class="[
              nota.JA_INCLUIDA == 1 ? 'bg-green-600 text-white' : 'hover:bg-indigo-50',
              'transition-colors'
            ]"
          >
            <td class="border px-3 py-1 text-center">
              <button
                @click="enviarNota(nota)"
                class="bg-indigo-600 hover:bg-indigo-700 disabled:bg-indigo-300 text-white font-semibold px-3 py-1 rounded transition"
                :disabled="nota.JA_INCLUIDA == 1"
              >
                Enviar
              </button>
            </td>

            <td class="border px-3 py-1">{{ nota.NOME_FORNECEDOR }}</td>
            <td class="border px-3 py-1">{{ formatDate(nota.DATA) }}</td>
             
            <td class="border px-3 py-1">{{ nota.NUMERO_NOTA }}</td>
            <td class="border px-3 py-1">{{ nota.EMPRESA }}</td>
            <td class="border px-3 py-1">{{ nota.COD_CLIENTE }}</td>
            <td class="border px-3 py-1">{{ formatDate(nota.DATA_NOTA) }}</td>
            <td class="border px-3 py-1 font-mono">R$ {{ nota.VALOR.toFixed(2) }}</td>
            <td class="border px-3 py-1">{{ nota.NATUREZA }}</td>
            <td class="border px-3 py-1 max-w-xs truncate" :title="nota.COMENTARIO">{{ truncate(nota.COMENTARIO, 30) }}</td>
            <td class="border px-3 py-1 font-semibold">{{ nota.STATUS_ENVIO }}</td>
            <td class="border px-3 py-1">{{ nota.RESULTADO_ENVIO }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Paginação -->
    <div class="max-w-xs mx-auto mt-6">
      <div class="flex justify-between items-center bg-white shadow rounded px-4 py-2">
        <button
          @click="paginaAtual--"
          :disabled="paginaAtual === 1"
          class="px-4 py-2 bg-indigo-200 text-indigo-700 rounded hover:bg-indigo-300 disabled:opacity-50 disabled:cursor-not-allowed transition"
        >
          Anterior
        </button>

        <span class="font-medium text-indigo-700">
          Página {{ paginaAtual }} de {{ totalPaginas }}
        </span>

        <button
          @click="paginaAtual++"
          :disabled="paginaAtual === totalPaginas"
          class="px-4 py-2 bg-indigo-200 text-indigo-700 rounded hover:bg-indigo-300 disabled:opacity-50 disabled:cursor-not-allowed transition"
        >
          Próxima
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const notas = ref([])
const loading = ref(true)
const error = ref(null)

const paginaAtual = ref(1)
const itensPorPagina = 10

const fetchNotas = async () => {
  try {
    const response = await axios.get('http://localhost:4343/listanotas')
    notas.value = response.data
  } catch (err) {
    error.value = err.message || 'Erro desconhecido'
  } finally {
    loading.value = false
  }
}

onMounted(fetchNotas)

const empresas = ref([
  { codigo: 2, nome: 'ORION KIA' },
  { codigo: 30, nome: 'TRUCKFOR' },
  { codigo: 20, nome: 'OGV' }
]);

const filtroGeral = ref('')
const filtroEmpresa = ref('')
const filtroFornecedor = ref('')
const filtroDataInicio = ref('')
const filtroDataFim = ref('')
const filtroStatus = ref('') // Inicializa com string vazia para "Todos os Status"

const notasFiltradas = computed(() => {
  const termoGeral = filtroGeral.value.toLowerCase().trim()

  return notas.value.filter(nota => {
    const empresaMatch = filtroEmpresa.value ? nota.EMPRESA === Number(filtroEmpresa.value) : true
    const fornecedorMatch = filtroFornecedor.value ? nota.NOME_FORNECEDOR.toLowerCase().includes(filtroFornecedor.value.toLowerCase()) : true

    let dataMatch = true
    if (filtroDataInicio.value) {
      dataMatch = new Date(nota.DATA_NOTA) >= new Date(filtroDataInicio.value)
    }
    if (dataMatch && filtroDataFim.value) {
      dataMatch = new Date(nota.DATA_NOTA) <= new Date(filtroDataFim.value)
    }

    // Lógica para o filtro de status:
    let statusMatch = true;
    if (filtroStatus.value !== '') { // Se um status específico foi selecionado
      if (filtroStatus.value === '0') { // Se o filtro é 'Pendente' (valor "0")
        // O status da nota deve ser 0 OU null (para Pendente)
        statusMatch = (nota.STATUS_ENVIO === 0 || nota.STATUS_ENVIO === null);
      } else {
        // Para 'Sucesso' (1) ou 'Erro' (2), compare diretamente (convertendo para string)
        statusMatch = String(nota.STATUS_ENVIO) === filtroStatus.value;
      }
    }
    // Se filtroStatus.value é '', então statusMatch permanece true (Todos os Status)


    if (!empresaMatch || !fornecedorMatch || !dataMatch || !statusMatch) return false

    if (!termoGeral) return true

    const textoConcat = `
      ${nota.NOME_FORNECEDOR}
      ${nota.COD_EMPRESA}
      ${nota.COD_CLIENTE}
      ${nota.ID_DOCUMENT}
      ${nota.NUMERO_NOTA}
      ${nota.DATA_NOTA}
      ${nota.NATUREZA}
      ${nota.COMENTARIO}
      ${nota.VALOR}
      ${nota.STATUS_ENVIO}
      ${nota.RESULTADO_ENVIO}
    `.toLowerCase()

    return textoConcat.includes(termoGeral)
  })


})

const totalPaginas = computed(() => Math.ceil(notasFiltradas.value.length / itensPorPagina))

const notasPaginadas = computed(() => {
  const inicio = (paginaAtual.value - 1) * itensPorPagina
  return notasFiltradas.value.slice(inicio, inicio + itensPorPagina)
})

const formatDate = (dateStr) => {
  const date = new Date(dateStr)
  return date.toLocaleDateString()
}

const truncate = (text, length) => {
  if (!text) return ''
  return text.length > length ? text.slice(0, length) + '...' : text
}

const enviarNota = async (nota) => {
  try {
    await axios.post('http://localhost:4343/sendNotaToNbs', [nota])
    alert(`Nota ${nota.NUMERO_NOTA} enviada com sucesso!`)
    await fetchNotas()
  } catch (err) {
    console.error('Erro ao enviar nota:', err)
    alert(`Erro ao enviar nota ${nota.NUMERO_NOTA}`)
  }
}


const notaBusca = ref('')

const buscarNota = async () => {
  if (!notaBusca.value.trim()) {
    alert('Digite o número da nota para buscar.')
    return
  }

  try {
    const response = await axios.post('http://localhost:4343/buscaNota', {
      nota: notaBusca.value.trim()
    })
    console.log('Resposta da nota buscada:', response.data)

    // Aqui você decide o que fazer com o resultado:
    // Exemplo: substituir as notas exibidas apenas por essa
    notas.value = Array.isArray(response.data) ? response.data : [response.data]
    paginaAtual.value = 1
  } catch (err) {
    console.error('Erro ao buscar nota:', err)
    alert('Erro ao buscar a nota. Verifique o número informado.')
  }
}

</script>

<style scoped>
/* Container principal */
.p-6 {
  padding: 1.5rem; /* equivalente a p-6 */
  max-width: 1100px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

/* Título */
h1 {
  font-size: 2rem; /* text-3xl */
  font-weight: bold;
  margin-bottom: 1.5rem;
  color: #222;
}

/* Container dos filtros com flex responsivo */
.filters-container {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  margin-bottom: 1.5rem;
}

/* Inputs */
input[type="text"],
input[type="number"],
input[type="date"] {
  padding: 0.5rem 0.75rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  flex-grow: 1;
  min-width: 150px;
  font-size: 1rem;
  box-sizing: border-box;
  transition: border-color 0.25s ease, box-shadow 0.25s ease;
}

input[type="text"]:focus,
input[type="number"]:focus,
input[type="date"]:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 6px rgba(59, 130, 246, 0.5);
}

/* Mensagens de loading e erro */
.text-gray-600 {
  color: #4b5563;
  font-style: italic;
}

.text-red-500 {
  color: #dc2626;
  font-weight: bold;
}

/* Tabela */
table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1rem;
  font-size: 0.9rem;
  box-shadow: 0 0 5px rgba(0,0,0,0.05);
  border-radius: 8px;
  overflow: hidden;
}

/* Cabeçalho da tabela */
thead tr {
  background-color: #f3f4f6;
  color: #374151;
  text-align: left;
  font-weight: 600;
  user-select: none;
}

/* Células */
th, td {
  border-bottom: 1px solid #e5e7eb;
  padding: 0.6rem 0.75rem;
  vertical-align: middle;
}

/* Coluna ações centralizada e largura fixa */
th:first-child,
td:first-child {
  text-align: center;
  width: 90px;
}

/* Linha hover */
tbody tr:hover {
  background-color: #f9fafb;
  transition: background-color 0.25s ease;
  cursor: pointer;
}

/* Linhas com nota já incluída */
tr.bg-incluida {
  background-color: #22c55e;
  color: white;
  font-weight: 600;
}

/* Botão Enviar */
button {
  background-color: #3b82f6;
  border: none;
  color: white;
  padding: 0.4rem 0.9rem;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
  font-size: 0.9rem;
}

button:hover:not(:disabled) {
  background-color: #2563eb;
}

button:disabled {
  background-color: #93c5fd;
  cursor: not-allowed;
  opacity: 0.7;
}

/* Paginação */
.pagination {
  max-width: 320px;
  margin-top: 1rem;
}

.pagination .flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.pagination button {
  background-color: #e5e7eb;
  color: #374151;
  padding: 0.4rem 1rem;
  border-radius: 6px;
  border: none;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.25s ease;
}

.pagination button:hover:not(:disabled) {
  background-color: #d1d5db;
}

.pagination button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pagination span {
  font-weight: 600;
  color: #374151;
}

/* Comentário com quebra de linha e largura limitada */
td.w-80 {
  max-width: 320px;
  white-space: pre-line;
  overflow-wrap: break-word;
}
</style>

