<template>
  <div class="app">
    <!-- Menú lateral estilo gym -->
    <aside class="sidebar">
      <h2>Pokémon Gym</h2>
      <img src="../public/007.png" alt="Charmander" class="sidebar-pokemon" />
      <nav>
        <button :class="{ active: view === 'users' }" @click="view = 'users'">Usuarios</button>
        <button :class="{ active: view === 'pokemon' }" @click="view = 'pokemon'">Pokémon</button>
      </nav>
    </aside>

    <!-- Contenido principal -->
    <main>
      <header>
        <h1 v-if="view === 'users'">Usuarios</h1>
        <h1 v-else>Pokémon</h1>
      </header>

      <!-- Loading -->
      <div v-if="loading" class="loading">✨ Cargando... ✨</div>

      <!-- Usuarios -->
      <section v-if="view === 'users' && !loading" class="cards-container">
        <div
            v-for="user in users"
            :key="user.id"
            class="card user-card"
            v-bind:style="{ animationDelay: `${0.1 * users.indexOf(user)}s` }"
        >
          <h3>{{ user.name }}</h3>
          <p>{{ user.email }}</p>
        </div>
        <button class="reload-btn" @click="fetchUsers">Recargar</button>
      </section>

      <!-- Pokémon -->
      <section v-if="view === 'pokemon' && !loading" class="pokemon-section">
        <div class="search">
          <input v-model="pokemonName" placeholder="Escribe un nombre" />
          <button @click="fetchPokemon">Buscar</button>
        </div>

        <div v-if="pokemon" class="cards-container">
          <div class="card pokemon-card" v-bind:style="{ animationDelay: '0.2s' }">
            <h3>{{ pokemon.name }}</h3>
            <img :src="pokemon.sprites.front_default" alt="imagen pokemon" />
            <p>Tipo: {{ pokemon.types.map(t => t.type.name).join(', ') }}</p>
          </div>
        </div>

        <div v-if="pokemonError" class="error-msg">❌ {{ pokemonError }} ❌</div>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { api } from "@/services/api.js"; // tu archivo de servicios

const view = ref("users");
const users = ref([]);
const pokemonName = ref("");
const pokemon = ref(null);
const loading = ref(false);
const pokemonError = ref("");

// --- Traer usuarios ---
async function fetchUsers() {
  loading.value = true;
  try {
    const res = await api.get("/users"); // coincide con Route::apiResource('users')
    users.value = res.data;
  } catch (error) {
    console.error("Error al traer usuarios:", error);
    alert("No se pudieron cargar los usuarios");
  } finally {
    loading.value = false;
  }
}

// --- Traer Pokémon por nombre ---
async function fetchPokemon() {
  if (!pokemonName.value) return;

  loading.value = true;
  pokemonError.value = "";
  pokemon.value = null;

  try {
    const res = await api.get(`/pokemon/${pokemonName.value.toLowerCase()}`);
    // coincide con Route::get('pokemon/{name}')
    pokemon.value = res.data;
  } catch (error) {
    console.error("Error al traer Pokémon:", error);
    pokemonError.value = "Pokémon no encontrado";
  } finally {
    loading.value = false;
  }
}

// Cargar usuarios al inicio
fetchUsers();
</script>

<style>
/* --- Layout --- */
.app {
  display: flex;
  min-height: 100vh;
  font-family: "Comic Sans MS", cursive;
  background: linear-gradient(to bottom, #ffe0f0, #ffd1e8);
  color: #a68fec;
}

/* --- Sidebar estilo gym --- */
.sidebar {
  width: 200px;
  background: #ffb6c1;
  padding: 20px;
  text-align: center;
  box-shadow: 2px 0 10px rgba(0,0,0,0.1);
}

.sidebar h2 {
  margin-bottom: 10px;
}

.sidebar-pokemon {
  width: 100px;
  margin-bottom: 20px;
}

.sidebar nav button {
  display: block;
  width: 100%;
  margin: 10px 0;
  padding: 10px;
  border-radius: 25px;
  border: none;
  background-color: rgba(118, 87, 222, 0.98);
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}
.sidebar nav button.active, .sidebar nav button:hover {
  background-color: deeppink;

}

/* --- Main content --- */
main {
  flex: 1;
  padding: 30px;
}

header h1 {
  font-size: 2em;
  margin-bottom: 20px;
}

/* --- Cards --- */
.cards-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}

.card {
  background: #fff0f8;
  border-radius: 20px;
  padding: 20px;
  width: 200px;
  text-align: center;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
  transition: transform 0.3s, box-shadow 0.3s;
  animation: fadeInUp 0.5s forwards;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0,0,0,0.3);
}

.user-card {
  background: #ffd1e8;
  border-radius: 25px;
  background-image: url("../public/pincel-pintura-rosa-textura-fondo_53876-102659.jpg");
  object-fit: cover;
}

.pokemon-card {
  background: #ffe4f0;
  border-radius: 20px;
  width: 250px;
  height: 250px;
  background-image: url("../public/fondo-acuarela-rosa-pintado-mano-detallado_1048-17039.jpg");

}

/* --- Animación cards --- */
@keyframes fadeInUp {
  0% { opacity: 0; transform: translateY(20px);}
  100% { opacity: 1; transform: translateY(0);}
}

/* --- Loading --- */
.loading {
  font-size: 1.2em;
  color: deeppink;
  margin: 20px 0;
  animation: blink 1s infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* --- Error Pokémon --- */
.error-msg {
  color: red;
  font-weight: bold;
  margin-top: 20px;
}

/* --- Botones --- */
.reload-btn, .search button {
  margin-top: 20px;
  padding: 10px 25px;
  border-radius: 25px;
  border: none;
  background-color: #291c3b;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}
.reload-btn:hover, .search button:hover {
  background-color: deeppink;
}

/* --- Input search --- */
.search {
  margin-bottom: 20px;
}

.search input {
  padding: 10px 15px;
  border-radius: 20px;
  border: 1px solid #ccc;
  width: 180px;
}

/* --- Pokémon image --- */
.pokemon-card img {
  width: 100px;
  margin-top: 10px;
}

/* --- Responsive --- */
@media (max-width: 768px) {
  .app {
    flex-direction: column;
  }
  .sidebar {
    width: 100%;
    box-shadow: none;
  }
  main {
    padding: 20px;
  }
}
</style>
