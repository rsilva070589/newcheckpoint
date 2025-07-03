<template>
  <Navbar class="w-full fixed top-0 left-0 z-50" style="margin-left: -30px; margin-right: -30px;" />
 
 

 <div v-if="store.Navbar==1">
  <Checkpoint />
 </div>

 <div v-if="store.Navbar==2">
  <Parametros />
 </div>


</template>

<script setup>
import { computed } from 'vue';
import { useStore } from '@/stores/store';

import Navbar from './views/Navbar.vue';
import ListaNotas from '../src/views/ListaNotas.vue';
import Checkpoint from '../src/views/Checkpoint.vue';
import Parametros from './views/Parametros.vue';

const store = useStore();

 store.Navbar=1

const viewAtual = computed(() => componentes[store.menuAtual]);
</script>
