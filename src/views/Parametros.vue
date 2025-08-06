<template>
  <div class="p-4 bg-gray-50 min-h-screen">
    <h2 class="text-3xl font-extrabold text-gray-800 mb-6 border-b-2 pb-2">
      Consulta de Tabelas
    </h2>

    <div
      class="flex flex-col md:flex-row md:items-center gap-4 mb-6 p-4 bg-white rounded-lg shadow-sm"
    >
      <div class="flex items-center gap-3">
        <label for="table-select" class="font-semibold text-gray-700"
          >Tabela:</label
        >
        <select
          id="table-select"
          v-model="selectedTable"
          class="border border-gray-300 px-4 py-2 rounded-md focus:ring-blue-500 focus:border-blue-500 transition duration-150 ease-in-out"
        >
          <option disabled value="">-- Escolha uma tabela --</option>
          <option
            v-for="(label, table) in tables"
            :key="table"
            :value="table"
          >
            {{ label }}
          </option>
        </select>
      </div>

      <div class="flex items-center gap-3">
        <label for="mesAno" class="font-semibold text-gray-700"
          >Mês/Ano:</label
        >
        <input
          id="mesAno"
          type="text"
          v-model="mesAno"
          placeholder="MM/YYYY"
          class="border border-gray-300 px-4 py-2 rounded-md w-36 focus:ring-blue-500 focus:border-blue-500 transition duration-150 ease-in-out"
        />
      </div>

      <button
        @click="fetchTable"
        :disabled="!selectedTable || !mesAnoValido"
        class="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 disabled:bg-gray-400 disabled:cursor-not-allowed transition duration-150 ease-in-out"
      >
        Carregar Dados
      </button>
    </div>

    <div
      v-if="selectedTable === 'NBS_API_AGENDAMENTOS' && editingRow"
      class="mb-6 p-6 bg-white rounded-lg shadow-md border-t-4 border-blue-500"
    >
      <h3 class="text-xl font-bold text-gray-800 mb-4">
        Editar Registro (NBS_API_AGENDAMENTOS)
      </h3>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-4">
        <div>
          <label for="edit-chassi" class="block text-sm font-medium text-gray-700"
            >CHASSI:</label
          >
          <input
            type="text"
            id="edit-chassi"
            v-model="editChassi"
            class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
        <div>
          <label for="edit-placa" class="block text-sm font-medium text-gray-700"
            >PLACA:</label
          >
          <input
            type="text"
            id="edit-placa"
            v-model="editPlaca"
            class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
        <div>
          <label
            for="edit-consultor"
            class="block text-sm font-medium text-gray-700"
            >CONSULTOR:</label
          >
          <input
            type="text"
            id="edit-consultor"
            v-model="editConsultor"
            class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
        <div>
          <label
            for="edit-cod-servico"
            class="block text-sm font-medium text-gray-700"
            >COD_SERVICO:</label
          >
          <input
            type="text"
            id="edit-cod-servico"
            v-model="editCodServico"
            class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
      </div>
      <div class="flex justify-end gap-3 mt-4">
        <button
          @click="cancelEdit"
          class="px-5 py-2 bg-gray-300 text-gray-800 rounded-md hover:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2 transition duration-150 ease-in-out"
        >
          Cancelar Edição
        </button>
        <button
          @click="saveEditedRow"
          class="px-5 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2 transition duration-150 ease-in-out"
        >
          Salvar Alterações
        </button>
      </div>
    </div>

    <div v-if="result.length" class="mb-4">
      <input
        type="text"
        v-model="filterText"
        placeholder="Filtrar linhas..."
        class="border border-gray-300 px-4 py-2 rounded-md w-full md:w-80 focus:ring-blue-500 focus:border-blue-500 transition duration-150 ease-in-out shadow-sm"
      />
    </div>

    <div v-if="paginatedData.length" class="bg-white rounded-lg shadow-md overflow-hidden">
      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-100">
            <tr>
              <th
                v-if="selectedTable === 'NBS_API_AGENDAMENTOS'"
                class="px-6 py-3 text-left text-xs font-medium text-gray-600 uppercase tracking-wider"
              >
                Ações
              </th>
              <th
                v-for="(value, key) in paginatedData[0]"
                :key="key"
                class="px-6 py-3 text-left text-xs font-medium text-gray-600 uppercase tracking-wider"
                :class="{
                  'observacao-column': key === 'OBSERVACAO',
                  'status-envio-column': key === 'status_envio'
                }"
                v-show="!columnsToHide.includes(key)"
              >
                {{ key }}
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-200">
            <tr v-for="(row, index) in paginatedData" :key="index">
              <td
                v-if="selectedTable === 'NBS_API_AGENDAMENTOS'"
                class="px-6 py-4 whitespace-nowrap text-sm text-gray-900"
              >
                <button
                  @click="openEditForm(row)"
                  class="text-blue-600 hover:text-blue-900 font-medium focus:outline-none focus:underline"
                >
                  Editar
                </button>
              </td>
              <td
                v-for="(value, key) in row"
                :key="key"
                class="px-6 py-4 text-sm text-gray-800 break-words whitespace-pre-wrap max-w-xs"
                :class="{
                  'observacao-column': key === 'OBSERVACAO',
                  'status-envio-column': key === 'status_envio'
                }"
                v-show="!columnsToHide.includes(key)"
              >
                <div class="flex flex-col gap-1">
                  <span class="leading-relaxed">
                    {{ truncateValue(formatValue(value)) }}
                    <span
                      v-if="formatValue(value).length > 100"
                      class="text-gray-400"
                      >...</span
                    >
                  </span>
                  <button
                    v-if="formatValue(value).length > 100"
                    @click="copyToClipboard(formatValue(value))"
                    class="text-blue-500 text-xs hover:text-blue-700 focus:outline-none focus:underline mt-1 self-start"
                  >
                    Copiar
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div
      v-if="result.length"
      class="flex justify-center items-center gap-4 mt-6 p-4 bg-white rounded-lg shadow-sm"
    >
      <button
        @click="prevPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 disabled:opacity-60 disabled:cursor-not-allowed transition duration-150 ease-in-out"
      >
        Anterior
      </button>
      <span class="text-gray-700"
        >Página <span class="font-semibold">{{ currentPage }}</span> de
        <span class="font-semibold">{{ totalPages }}</span></span
      >
      <button
        @click="nextPage"
        :disabled="currentPage === totalPages"
        class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 disabled:opacity-60 disabled:cursor-not-allowed transition duration-150 ease-in-out"
      >
        Próximo
      </button>
    </div>

    <div v-else class="text-gray-600 text-center mt-8 p-4 bg-white rounded-lg shadow-sm">
      Nenhum dado carregado. Por favor, selecione uma tabela e um Mês/Ano.
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import axios from 'axios'

const tables = {
  'Hangfire_Renault_2.dbo.GM_SFCRM_RECEIVEID': 'Recebimentos GM_SFCRM_RECEIVEID',
  'NBS_API_AGENDAMENTOS': 'Logs NBS_API_AGENDAMENTOS',
  'Hangfire_Renault_2.dbo.leads_received': 'Renault CRMI Fase1',
  'Hangfire_Renault_2.dbo.leads_send': 'Renault CRMI Fase3 - Leads Rede',
  'Hangfire_Renault_2.dbo.facebook_leads_webhook': 'Facebook',
  'Hangfire_Renault_2.dbo.send_BancoRenovec_mapsis': 'Banco RenovAction',

}

const selectedTable = ref('')
const mesAno = ref('')
const result = ref([])
const filterText = ref('')

const columnsToHide = ['endereco', 'cep', 'cidade', 'estado','status']

// Refs for edit form
const editingRow = ref(null) // Stores the data of the row being edited
const editChassi = ref('')
const editPlaca = ref('')
const editConsultor = ref('')
const editCodServico = ref('')

const mesAnoValido = computed(() => {
  const regex = /^(0[1-9]|1[0-2])\/\d{4}$/
  return regex.test(mesAno.value)
})

const fetchTable = async () => {
  const [mesStr, anoStr] = mesAno.value.split('/')
  const mes = parseInt(mesStr)
  const ano = parseInt(anoStr)

  try {
    const { data } = await axios.post(
      'https://api.mapsis.com.br/apiexpress/monitor/table',
      {
        tableName: selectedTable.value,
        mes,
        ano
      },
      { headers: { 'Content-Type': 'application/json' } }
    )
    result.value = Array.isArray(data.results) ? data.results : []
    currentPage.value = 1
    filterText.value = ''
    editingRow.value = null; // Clear any active editing when new data is loaded
  } catch (error) {
    console.error(error)
    alert('Erro ao carregar dados. Verifique o console para mais detalhes.')
  }
}

// Functions for edit form
const openEditForm = (row) => {
  // Only open form if the selected table is NBS_API_AGENDAMENTOS
  if (selectedTable.value !== 'NBS_API_AGENDAMENTOS') {
    return;
  }
  editingRow.value = { ...row }; // Create a copy to avoid directly modifying table data
  editChassi.value = row.CHASSI || '';
  editPlaca.value = row.PLACA || '';
  editConsultor.value = row.CONSULTOR || '';
  editCodServico.value = row.COD_SERVICO || '';
};

const cancelEdit = () => {
  editingRow.value = null; // Clear editing state
  editChassi.value = '';
  editPlaca.value = '';
  editConsultor.value = '';
  editCodServico.value = '';
};

const saveEditedRow = async () => {
  if (!editingRow.value || !editingRow.value.ID) {
    alert('Erro: Não foi possível identificar o registro para atualização. ID do registro ausente.');
    return;
  }

  try {
    const payload = {
      tableName: selectedTable.value,
      updateFields: {
        CHASSI: editChassi.value,
        PLACA: editPlaca.value,
        CONSULTOR: editConsultor.value,
        COD_SERVICO: editCodServico.value,
        descricao_erro: '' // Clear this field as requested
      },
      whereConditions: {
        ID: editingRow.value.ID
      }
    };

    await axios.post(
      'https://api.mapsis.com.br/apiexpress/monitor/update',
      payload,
      { headers: { 'Content-Type': 'application/json' } }
    );

    alert('Registro atualizado com sucesso!');
    cancelEdit(); // Close the edit form
    // Re-fetch the table data to reflect the changes
    fetchTable();
  } catch (error) {
    console.error('Erro ao salvar:', error);
    alert('Erro ao salvar dados: ' + (error.response?.data?.message || error.message || 'Verifique o console para detalhes.'));
  }
};


const formatValue = (value) => {
  if (value === null || value === undefined) return ''
  if (typeof value === 'object') return JSON.stringify(value, null, 2)
  return String(value)
}

const truncateValue = (value) => {
  return value.length > 100 ? value.slice(0, 100) : value
}

const copyToClipboard = (value) => {
  navigator.clipboard.writeText(value)
    .then(() => alert('Copiado com sucesso!'))
    .catch(() => alert('Erro ao copiar'))
}

const filteredData = computed(() => {
  if (!filterText.value) return result.value
  const lowerFilter = filterText.value.toLowerCase()
  return result.value.filter(row =>
    Object.values(row).some(val =>
      String(val).toLowerCase().includes(lowerFilter)
    )
  )
})

const itemsPerPage = 4
const currentPage = ref(1)

const totalPages = computed(() => {
  return Math.max(1, Math.ceil(filteredData.value.length / itemsPerPage))
})

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredData.value.slice(start, end)
})

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}

// Watch for table changes to reset editing state
watch(selectedTable, () => {
  cancelEdit();
});
</script>

<style scoped>
/* Base table styles */
table {
  border-collapse: collapse;
  width: 100%;
  font-size: 0.875rem; /* smaller font */
  table-layout: auto; /* Allows columns to adjust width based on content */
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px 12px;
  vertical-align: top;
  word-break: break-word;
  min-width: 100px; /* Minimum width for columns, adjust as needed */
  white-space: pre-wrap; /* line break within cell */
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  text-align: left;
}

/* Specific styling for the OBSERVACAO column */
.observacao-column {
  min-width: 300px; /* Give it a larger minimum width */
  width: 25%; /* Try to allocate a percentage of the table width */
}

/* Specific styling for the status_envio column */
.status-envio-column {
  min-width: 200px; /* Give it a larger minimum width than others, but perhaps less than OBSERVACAO */
  width: 15%; /* Try to allocate a percentage of the table width */
}

thead th {
  background-color: #f3f4f6; /* light gray */
  font-weight: 600;
  color: #374151; /* dark gray */
}

tbody tr:nth-child(even) {
  background-color: #fafafa;
}

tbody tr:hover {
  background-color: #f0f9ff;
}

button {
  background: none;
  border: none;
  padding: 0;
  color: #2563eb;
  font-size: 0.8rem;
  cursor: pointer;
}

button:hover {
  text-decoration: underline;
}

.overflow-x-auto {
  overflow-x: auto; /* Ensures horizontal scrollbar */
  -webkit-overflow-scrolling: touch;
}

/* Modal specific styles */
.fixed {
  z-index: 1000; /* Ensure modal is on top of other content */
}
</style>