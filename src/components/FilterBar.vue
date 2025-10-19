<script setup>
import { ref, watch, computed } from 'vue'
const props = defineProps({ cars: { type: Array, required: true }})
const emit = defineEmits(['update:filters','clear'])

const marca = ref(''), modelo = ref(''), anio = ref(''), color = ref(''), precioMin = ref(''), precioMax = ref('')

const marcas  = computed(() => [...new Set(props.cars.map(c=>c.marca))].sort())
const modelos = computed(() => (marca.value ? props.cars.filter(c=>c.marca===marca.value) : props.cars)
  .reduce((s,c)=>s.add(c.modelo), new Set()))
const colores = computed(() => [...new Set(props.cars.map(c=>c.color))].sort())
const anios   = computed(() => [...new Set(props.cars.map(c=>c.anio))].sort((a,b)=>b-a))

watch([marca,modelo,anio,color,precioMin,precioMax], () => {
  emit('update:filters', {
    marca: marca.value || null,
    modelo: modelo.value || null,
    anio: anio.value ? Number(anio.value) : null,
    color: color.value || null,
    precioMin: precioMin.value ? Number(precioMin.value) : null,
    precioMax: precioMax.value ? Number(precioMax.value) : null
  })
})

function limpiar(){ marca.value=modelo.value=anio.value=color.value=''; precioMin.value=precioMax.value=''; emit('clear') }
</script>

<template>
  <div class="card">
    <div class="row" style="align-items:end">
      <div>
        <label class="small">Marca</label>
        <select class="select" v-model="marca">
          <option value="">Todas</option>
          <option v-for="m in marcas" :key="m" :value="m">{{ m }}</option>
        </select>
      </div>
      <div>
        <label class="small">Modelo</label>
        <select class="select" v-model="modelo">
          <option value="">Todos</option>
          <option v-for="m in Array.from(modelos)" :key="m" :value="m">{{ m }}</option>
        </select>
      </div>
      <div>
        <label class="small">Año</label>
        <select class="select" v-model="anio">
          <option value="">Todos</option>
          <option v-for="a in anios" :key="a" :value="a">{{ a }}</option>
        </select>
      </div>
      <div>
        <label class="small">Color</label>
        <select class="select" v-model="color">
          <option value="">Todos</option>
          <option v-for="c in colores" :key="c" :value="c">{{ c }}</option>
        </select>
      </div>
      <div>
        <label class="small">Precio mínimo</label>
        <input class="input" type="number" min="0" step="1000" v-model="precioMin" placeholder="250000"/>
      </div>
      <div>
        <label class="small">Precio máximo</label>
        <input class="input" type="number" min="0" step="1000" v-model="precioMax" placeholder="500000"/>
      </div>
    </div>

    <div style="display:flex; gap:10px; margin-top:12px">
      <button class="btn">Buscar vehículos</button>
      <button class="btn" style="background:#22313f; color:#b8c5d1" @click="limpiar">Limpiar</button>
    </div>
  </div>
</template>
