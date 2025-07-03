<template>
    <div class="container">
    
  
      <div v-if="!selecionado">
        <table class="gridview-table">
          <thead>
            <tr>
              <th>Código Tipo - Categoria</th>
              <th>Quantidade</th>
              <th>Ações</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(items, chave, index) in agrupados"
              :key="chave"
              :class="index % 2 === 0 ? 'even' : 'odd'"
              @click="selecionarDetalhe(chave, items[0]?.COD_TIPO)"
            >
              <td>{{ chave }}</td>
              <td class="center">{{ items.length }}</td>
              <td class="center">
                <button class="btn" @click.stop="selecionarDetalhe(chave, items[0]?.COD_TIPO)">Ver detalhes →</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  
      <div v-else>
        <CheckpointDetalhes
          :titulo="selecionado"
          :codigoTipo="codigoTipoSelecionado"
          :itens="agrupados[selecionado]"
          @voltar="voltarLista"
        />
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  import axios from 'axios'
  import CheckpointDetalhes from './CheckpointDetalhes.vue'
  
  const dados = ref([])
  const agrupados = ref({})
  const selecionado = ref(null)
  const codigoTipoSelecionado = ref(null)
  
  const carregarDados = async () => {
    try {
      const response = await axios.get('https://api.mapsis.com.br/apiexpress/monitor')
      dados.value = response.data.results || []
  
      const agrupamento = {}
      for (const item of dados.value) {
        const chave = `${item.COD_TIPO} - ${item.CATEGORIA}`
        if (!agrupamento[chave]) agrupamento[chave] = []
        agrupamento[chave].push(item)
      }
      agrupados.value = agrupamento
  
    } catch (error) {
      console.error('Erro ao buscar dados:', error)
    }
  }
  
  const selecionarDetalhe = (chave, codigoTipo) => {
    selecionado.value = chave
    codigoTipoSelecionado.value = codigoTipo
  }
  
  const voltarLista = () => {
    selecionado.value = null
    codigoTipoSelecionado.value = null
  }
  
  onMounted(carregarDados)
  </script>
  
  <style scoped>
  .container {
    max-width: 900px;
    margin: 2rem auto;
    font-family: Arial, sans-serif;
  }
  
  h1 {
    font-size: 2rem;
    margin-bottom: 1.5rem;
    color: #1e293b;
    text-align: center;
  }
  
  .gridview-table {
    width: 100%;
    border-collapse: collapse;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 8px rgb(0 0 0 / 0.1);
    cursor: default;
  }
  
  .gridview-table thead {
    background-color: #2563eb;
    color: white;
    user-select: none;
  }
  
  .gridview-table th,
  .gridview-table td {
    padding: 12px 16px;
    border-bottom: 1px solid #e5e7eb;
    text-align: left;
  }
  
  .gridview-table th {
    font-weight: 600;
    text-align: left;
  }
  
  .gridview-table tbody tr.even {
    background-color: #f9fafb;
  }
  
  .gridview-table tbody tr.odd {
    background-color: #ffffff;
  }
  
  .gridview-table tbody tr:hover {
    background-color: #e0e7ff;
  }
  
  .gridview-table tbody td.center {
    text-align: center;
  }
  
  .btn {
    background-color: transparent;
    border: none;
    color: #2563eb;
    font-weight: 600;
    cursor: pointer;
    padding: 0;
    font-size: 1rem;
  }
  
  .btn:hover {
    text-decoration: underline;
    color: #1e40af;
  }
  </style>
  