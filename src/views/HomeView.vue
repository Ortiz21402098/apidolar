<template>
  <h1>🎬 Cartelera de Cine</h1>
  <p class="subtitulo">Películas disponibles en cartelera</p>

  <!-- Estado de carga -->
  <p v-if="cargando" class="cargando">Cargando películas...</p>

  <!-- Mensaje de error si falla la petición -->
  <p v-else-if="error" class="mensaje-error">{{ error }}</p>

  <!-- Grilla de películas cuando ya hay datos -->
  <div v-else id="peliculas">
    <div v-for="pelicula in peliculas" :key="pelicula.titulo" class="card-pelicula">
      <img :src="pelicula.imagen" :alt="pelicula.titulo" />

      <div class="card-contenido">
        <h2>{{ pelicula.titulo }}</h2>
        <p class="etiqueta">{{ pelicula.genero }}</p>
        <p class="anio">{{ pelicula.año }}</p>
        <p class="descripcion">{{ pelicula.descripcion }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

// Variables reactivas para manejar los datos, carga y errores
const peliculas = ref([])
const cargando = ref(true)
const error = ref(null)

// Función asincrónica para obtener los datos
const obtenerPeliculas = async () => {
  try {
    const respuesta = await fetch('./peliculas.json')
    const datos = await respuesta.json()
    peliculas.value = datos
  } catch (err) {
    error.value = err.message
  } finally {
    cargando.value = false
  }
}

// Ejecutar la función asincrónica al montar el componente
onMounted(() => {
  obtenerPeliculas()
})
</script>

<style scoped>
/* Reset y estilos generales */
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

#peliculas {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
    gap: 2rem;
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
}

h1 {
    font-size: 2.2rem;
    margin-bottom: 0.5rem;
    color: #e50914;
    text-align: center;
}

p.subtitulo {
    color: #a3a3a3;
    margin-bottom: 1.5rem;
    font-size: 1.1rem;
    text-align: center;
}

/* Mensajes de carga y error */
.cargando {
    color: #ffffff;
    text-align: center;
    font-size: 1.2rem;
}

.mensaje-error {
    margin-top: 1.5rem;
    color: #ff4d4d;
    font-weight: bold;
    font-size: 1.1rem;
    text-align: center;
}

/* Tarjeta individual de película */
.card-pelicula {
    background-color: #1f1f1f;
    border-radius: 10px;
    overflow: hidden;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    transition: transform 0.3s, box-shadow 0.3s;
}

.card-pelicula:hover {
    transform: translateY(-6px);
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.7);
}

.card-pelicula img {
    width: 100%;
    height: 320px;
    object-fit: cover;
    background-color: #2b2b2b;
}

.card-contenido {
    padding: 1.2rem;
    display: flex;
    flex-direction: column;
    flex-grow: 1;
}

.card-contenido h2 {
    font-size: 1.25rem;
    margin-bottom: 0.5rem;
    color: #ffffff;
}

.etiqueta {
    font-size: 0.85rem;
    color: #e50914;
    font-weight: bold;
    margin-bottom: 0.2rem;
}

.anio {
    font-size: 0.85rem;
    color: #8c8c8c;
    margin-bottom: 0.8rem;
}

.descripcion {
    font-size: 0.9rem;
    color: #cccccc;
    line-height: 1.4;
    flex-grow: 1;
}
</style>

