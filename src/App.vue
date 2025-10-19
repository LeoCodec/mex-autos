<script setup>
import { ref, computed, onMounted } from 'vue'
import FilterBar from './components/FilterBar.vue'
import CarCard from './components/CarCard.vue'

const cars = ref([])
const filters = ref({})
const loading = ref(true)
const errorMsg = ref('')

onMounted(async () => {
  try {
    const res = await fetch('/data/cars.json')
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    cars.value = await res.json()
  } catch (err) {
    errorMsg.value = 'No se pudo cargar el catálogo. Revisa /public/data/cars.json'
    console.error(err)
  } finally {
    loading.value = false
  }
})

function setFilters(f){ filters.value = f }
function clearFilters(){ filters.value = {} }

const filtered = computed(() => {
  const f = filters.value
  const match = (val, term) => !term || String(val ?? '').toLowerCase().includes(String(term).toLowerCase())
  const inRange = (num, min, max) =>
    (num == null) || ((min == null || num >= min) && (max == null || num <= max))

  return cars.value.filter(c =>
    match(c.marca,  f.marca) &&
    match(c.modelo, f.modelo) &&
    match(c.color,  f.color) &&
    (f.anio ? c.anio === f.anio : true) &&
    inRange(c.precio, f.precioMin, f.precioMax)
  )
})
</script>

<template>
  <!-- HEADER -->
  <header class="header">
    <nav class="navbar">
      <div class="logo">
        <img src="/src/assets/logo.svg" alt="MexAutos" style="height:28px;vertical-align:middle;margin-right:8px" />
        MexAutos
      </div>
      <ul class="nav-links">
        <li><a href="#">Catálogo</a></li>
        <li><a href="#">Financiamiento</a></li>
        <li><a href="#">Contacto</a></li>
      </ul>
    </nav>
  </header>

  <!-- HERO -->
  <!-- Si no tienes la imagen, este estilo inline deja solo el degradado (se ve bien igual) -->
  <section
    class="hero"
    style="background: linear-gradient(rgba(0,34,77,.85), rgba(93,14,65,.85)), url('/assets/hero-bg.jpg') center/cover;"
  >
    <h1>Encuentra tu auto perfecto</h1>
    <p>Explora nuestro catálogo por marca, modelo, año, color y precio.</p>
  </section>

  <!-- FILTROS -->
  <section class="filter-section">
    <FilterBar
      :cars="cars"
      @update:filters="setFilters"
      @clear="clearFilters"
    />
  </section>

  <!-- ESTADOS -->
  <div style="max-width:1200px;margin:0 auto;padding:0 2rem">
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
    </div>
    <p v-else class="small">{{ filtered.length }} resultado(s)</p>
    <p v-if="errorMsg" class="no-results" style="margin-top:1rem">{{ errorMsg }}</p>
  </div>

  <!-- GRID DE AUTOS -->
  <section class="cars-grid" v-if="!loading && !errorMsg && filtered.length">
    <CarCard v-for="(car, i) in filtered" :key="i" :car="car" />
  </section>

  <!-- SIN RESULTADOS -->
  <div v-if="!loading && !errorMsg && !filtered.length" class="no-results">
    <h3>Sin resultados</h3>
    <p>Prueba modificando los filtros (marca, modelo, año, color o precio).</p>
  </div>

  <!-- FOOTER -->
  <footer class="footer">
    © 2025 MexAutos — Todos los derechos reservados
  </footer>
</template>
