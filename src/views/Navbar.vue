<template>
  <nav class="navbar">
    <div class="navbar-container">
      <div class="navbar-logo"> Checkpoints</div>
      <ul class="navbar-links">
        <li><a href="#" @click="store.Navbar=1">Checkpoint</a></li>
        <li><a href="#" @click="store.Navbar=2">Logs</a></li>
        <li><a href="#" @click="store.Navbar=3">Renovec</a></li>
        <li><a href="#" @click="store.Navbar=4">Upload</a></li>
      </ul>
      <button class="navbar-toggle" @click="toggleMenu" aria-label="Menu">
        ☰
      </button>
    </div>

    <ul v-if="menuOpen" class="navbar-links-mobile">
      <li><a href="#" @click="store.Navbar=1">Checkpoint</a></li>
      <li><a href="#" @click="store.Navbar=2">Logs</a></li>
      <li><a href="#" @click="store.Navbar=3">Renovec</a></li>
      <li><a href="#" @click="store.Navbar=4">Upload</a></li>
    </ul>
  </nav>
</template>

<script setup>
import { useStore } from '@/stores/store';
import { ref } from 'vue';

const store = useStore();
const menuOpen = ref(false);

const toggleMenu = () => {
  menuOpen.value = !menuOpen.value;
};

</script>

<style scoped>
.navbar {
  background-color: #3b82f6; /* azul */
  color: white;
  padding: 0.75rem 1.5rem;
  font-family: Arial, sans-serif;
  user-select: none;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}

/* Container para alinhar itens e limitar largura */
.navbar-container {
  max-width: 1100px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* Logo */
.navbar-logo {
  font-weight: 700;
  font-size: 1.4rem;
  cursor: default;
}

/* Links desktop */
.navbar-links {
  list-style: none;
  display: flex;
  gap: 1.2rem;
}

.navbar-links li a {
  color: white;
  text-decoration: none;
  font-weight: 600;
  padding: 0.3rem 0.5rem;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.navbar-links li a:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

/* Botão para abrir menu mobile - escondido no desktop */
.navbar-toggle {
  display: none;
  font-size: 1.8rem;
  background: none;
  border: none;
  color: white;
  cursor: pointer;
}

/* Menu mobile (aparece ao clicar no botão) */
.navbar-links-mobile {
  background-color: #3b82f6;
  list-style: none;
  padding: 0.75rem 1.5rem;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.navbar-links-mobile li a {
  color: white;
  font-weight: 600;
  text-decoration: none;
  padding: 0.5rem 0.75rem;
  border-radius: 4px;
  display: block;
  transition: background-color 0.3s ease;
}

.navbar-links-mobile li a:hover {
  background-color: rgba(255, 255, 255, 0.2);
}

/* Responsividade */
@media (max-width: 768px) {
  .navbar-links {
    display: none; /* Esconder links no desktop */
  }

  .navbar-toggle {
    display: block; /* Mostrar botão hamburguer */
  }
}
</style>
