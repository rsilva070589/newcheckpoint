<template>
  <Navbar class="w-full fixed top-0 left-0 z-50" style="margin-left: -30px; margin-right: -30px;" />
 
 

 <div v-if="store.Navbar==1">
  <Checkpoint />
 </div>

 <div v-if="store.Navbar==2">
  <Parametros />
 </div>

 <div v-if="store.Navbar==3">
  <Renovec />
 </div>

 <div v-if="store.Navbar==4">
  <UploadRenovec />
 </div>

</template>

<script setup>
import { computed } from 'vue';
import { useStore } from '@/stores/store';

import Navbar from './views/Navbar.vue';
import ListaNotas from '../src/views/ListaNotas.vue';
import Checkpoint from '../src/views/Checkpoint.vue';
import Parametros from './views/Parametros.vue';
import Renovec from './views/Renovec.vue';
import UploadRenovec from './views/UploadRenovec.vue';

const store = useStore();

 store.Navbar=1

const viewAtual = computed(() => componentes[store.menuAtual]);
</script>
