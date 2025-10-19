<script setup>
import { ref, computed, onMounted } from 'vue'
import FilterBar from './components/FilterBar.vue'
import CarCard from './components/CarCard.vue'

const cars = ref([])
const filters = ref({})
const loading = ref(true)
const errorMsg = ref('')
const datasetActivo = ref('americanos') // 'americanos', 'asiaticos', 'todos'

// Cargar dataset al iniciar
onMounted(async () => {
  await cargarDataset('americanos')
})

// Función para cambiar entre datasets
async function cargarDataset(tipo) {
  loading.value = true
  errorMsg.value = ''
  datasetActivo.value = tipo
  filters.value = {} // Limpiar filtros al cambiar dataset
  
  try {
    let datos = []
    
    if (tipo === 'americanos') {
      const res = await fetch('/data/carsamericans.json')
      if (!res.ok) throw new Error(`HTTP ${res.status} - No se encontró carsamericans.json`)
      datos = await res.json()
    } 
    else if (tipo === 'asiaticos') {
      const res = await fetch('/data/carsasia.json')
      if (!res.ok) throw new Error(`HTTP ${res.status} - No se encontró carsasia.json`)
      datos = await res.json()
    }
    else if (tipo === 'todos') {
      // Cargar ambos datasets
      const [resAmericanos, resAsiaticos] = await Promise.all([
        fetch('/data/carsamericans.json'),
        fetch('/data/carsasia.json')
      ])
      
      if (!resAmericanos.ok || !resAsiaticos.ok) {
        throw new Error('No se pudieron cargar uno o ambos catálogos')
      }
      
      const [autosAmericanos, autosAsiaticos] = await Promise.all([
        resAmericanos.json(),
        resAsiaticos.json()
      ])
      
      datos = [...autosAmericanos, ...autosAsiaticos]
    }
    
    cars.value = datos
    console.log(`✅ Cargados ${datos.length} autos (${tipo})`)
    
  } catch (err) {
    console.error('❌ Error cargando dataset:', err)
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

// Computed para autos filtrados
const filtered = computed(() => {
  const f = filters.value
  console.log('🔍 Filtros activos:', f)
  
  const match = (val, term) => {
    if (!term) return true
    const valStr = String(val ?? '').toLowerCase()
    const termStr = String(term).toLowerCase()
    return valStr.includes(termStr)
  }
  
  const inRange = (num, min, max) =>
    (num == null) || ((min == null || num >= min) && (max == null || num <= max))

  const resultados = cars.value.filter(c =>
    match(c.marca,  f.marca) &&
    match(c.modelo, f.modelo) &&
    match(c.color,  f.color) &&
    match(c.origen, f.origen) &&
    (f.anio ? c.anio === f.anio : true) &&
    inRange(c.precio, f.precioMin, f.precioMax)
  )
  
  console.log(`📊 Resultados: ${resultados.length} de ${cars.value.length}`)
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
      <p>Cargando catálogo {{ datasetActivo }}...</p>
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
      Prueba modificando los filtros o selecciona otra categoría.
    </p>
    <button @click="clearFilters()" class="btn btn-primary" style="margin-top: 1.5rem;">
      🔄 Limpiar todos los filtros
    </button>
  </div>

  <!-- SIN DATOS -->
  <div v-if="!loading && !errorMsg && cars.length === 0" class="no-results">
    <h3>📭 Catálogo vacío</h3>
    <p>No hay autos disponibles en esta categoría.</p>
    <p style="margin-top: 0.5rem; font-size: 0.9rem; color: var(--gray);">
      Selecciona otra categoría o verifica los archivos de datos.
    </p>
  </div>

  <!-- FOOTER -->
  <footer class="footer">
    © 2025 MexAutos — Concesionaria especializada en autos americanos y asiáticos
  </footer>
</template>

<style scoped>
/* Estilos adicionales específicos para App.vue */
.result-count {
  color: var(--gray-700);
  font-weight: 500;
  font-size: 1rem;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 200px;
  flex-direction: column;
  gap: 1rem;
  color: var(--gray-500);
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid var(--gray-200);
  border-top: 4px solid var(--primary);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.no-results {
  text-align: center;
  padding: 4rem 2rem;
  color: var(--gray-500);
  max-width: 600px;
  margin: 0 auto;
}

.no-results h3 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: var(--gray-700);
  font-weight: 600;
}
</style>