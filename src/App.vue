[file name]: App.vue
[file content begin]
<script setup>
import { ref, computed, onMounted } from 'vue'
import FilterBar from './components/FilterBar.vue'
import CarCard from './components/CarCard.vue'

const cars = ref([])
const filters = ref({})
const loading = ref(true)
const errorMsg = ref('')

// Cargar TODOS los datasets al iniciar
onMounted(async () => {
  await cargarTodosLosAutos()
})

// Función para cargar TODOS los autos
async function cargarTodosLosAutos() {
  loading.value = true
  errorMsg.value = ''
  filters.value = {}
  
  try {
    // Cargar ambos datasets
    const [resAmericanos, resAsiaticos] = await Promise.all([
      fetch('/data/carsamericans.json'),
      fetch('/data/carsasia.json')
    ])
    
    if (!resAmericanos.ok || !resAsiaticos.ok) {
      throw new Error('No se pudieron cargar los catálogos')
    }
    
    const [autosAmericanos, autosAsiaticos] = await Promise.all([
      resAmericanos.json(),
      resAsiaticos.json()
    ])
    
    // Combinar todos los autos
    cars.value = [...autosAmericanos, ...autosAsiaticos]
    console.log(`✅ Cargados ${cars.value.length} autos (americanos + asiáticos)`)
    
  } catch (err) {
    console.error('❌ Error cargando datasets:', err)
    errorMsg.value = `Error: ${err.message}`
    cars.value = []
  } finally {
    loading.value = false
  }
}

// Manejar filtros
function setFilters(f){ 
  filters.value = f 
}

function clearFilters(){ 
  filters.value = {} 
}

// Computed para autos filtrados - CORREGIDO
const filtered = computed(() => {
  const f = filters.value
  console.log('🔍 Filtros activos:', f)
  
  // Función para comparación exacta
  const matchExact = (val, term) => {
    if (!term) return true
    return String(val).toLowerCase() === String(term).toLowerCase()
  }
  
  // Función para rango de precios
  const inRange = (num, min, max) =>
    (num == null) || ((min == null || num >= min) && (max == null || num <= max))

  // Aplicar filtros
  const resultados = cars.value.filter(c => {
    return (
      (!f.marca || c.marca.toLowerCase().includes(f.marca.toLowerCase())) &&
      (!f.modelo || c.modelo.toLowerCase().includes(f.modelo.toLowerCase())) &&
      (!f.color || c.color.toLowerCase().includes(f.color.toLowerCase())) &&
      (!f.origen || c.origen === f.origen) && // ← Comparación EXACTA para origen
      (!f.anio || c.anio === f.anio) &&
      inRange(c.precio, f.precioMin, f.precioMax)
    )
  })
  
  console.log(`📊 Resultados: ${resultados.length} de ${cars.value.length}`)
  
  // DEBUG: Mostrar qué autos tienen cada origen
  if (f.origen) {
    const autosConOrigen = cars.value.filter(c => c.origen === f.origen)
    console.log(`🚗 Autos con origen "${f.origen}":`, autosConOrigen.length)
    console.log('📝 Ejemplos:', autosConOrigen.slice(0, 3))
  }
  
  return resultados
})
</script>

<template>
  <!-- HEADER -->
  <header class="header">
    <nav class="navbar">
      <div class="logo">
        <img src="/src/assets/logo.png" alt="MexAutos" style="height:28px;vertical-align:middle;margin-right:8px" />
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
  <section class="hero">
    <h1>Encuentra tu auto perfecto</h1>
    <p>Explora nuestro catálogo premium de vehículos americanos y asiáticos</p>
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
      <p>Cargando catálogo completo...</p>
    </div>
    
    <div v-else style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem;">
      <p class="result-count">
        <strong>{{ filtered.length }}</strong> resultado(s) de 
        <strong>{{ cars.length }}</strong> autos en catálogo
      </p>
      <button 
        v-if="filtered.length < cars.length" 
        @click="clearFilters"
        class="btn btn-secondary"
        style="padding: 0.5rem 1rem; font-size: 0.9rem;"
      >
        🔄 Limpiar filtros
      </button>
    </div>
    
    <p v-if="errorMsg" class="error" style="background: #fee; color: #c53030; padding: 1rem; border-radius: 8px; border: 1px solid #feb2b2;">
      {{ errorMsg }}
    </p>
  </div>

  <!-- GRID DE AUTOS -->
  <section class="cars-grid" v-if="!loading && !errorMsg && filtered.length">
    <CarCard 
      v-for="(car, i) in filtered" 
      :key="`${car.marca}-${car.modelo}-${i}`" 
      :car="car" 
    />
  </section>

  <!-- SIN RESULTADOS -->
  <div v-if="!loading && !errorMsg && !filtered.length && cars.length > 0" class="no-results">
    <h3>😔 Sin resultados</h3>
    <p>No encontramos autos que coincidan con tus filtros.</p>
    <p style="margin-top: 0.5rem; font-size: 0.9rem; color: var(--gray);">
      Prueba modificando los filtros.
    </p>
    <button @click="clearFilters()" class="btn btn-primary" style="margin-top: 1.5rem;">
      🔄 Limpiar todos los filtros
    </button>
  </div>

  <!-- SIN DATOS -->
  <div v-if="!loading && !errorMsg && cars.length === 0" class="no-results">
    <h3>📭 Catálogo vacío</h3>
    <p>No hay autos disponibles.</p>
    <p style="margin-top: 0.5rem; font-size: 0.9rem; color: var(--gray);">
      Verifica los archivos de datos.
    </p>
  </div>

  <!-- FOOTER -->
  <footer class="footer">
    © 2025 MexAutos — Concesionaria especializada en autos americanos y asiáticos
  </footer>
</template>

<style scoped>
/* Tus estilos existentes... */
</style>
[file content end]