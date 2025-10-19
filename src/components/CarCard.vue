<template>
  <article class="car-card">
    <div class="car-image-container">
      <img
        :src="getCarImage(car)"
        :alt="`${car.marca} ${car.modelo}`"
        class="car-image"
        @error="handleImageError"
      />
      <span class="year-badge">{{ car.anio || 'N/A' }}</span>
      <div class="image-overlay">
        <button class="quick-view-btn">Vista rápida</button>
      </div>
    </div>

    <div class="car-content">
      <div class="car-header">
        <h3 class="car-title">{{ car.marca }} {{ car.modelo }}</h3>
        <div class="car-price">
          {{ formatPrice(car.precio) }}
        </div>
      </div>

      <div class="car-details">
        <div class="detail-item">
          <span class="detail-label">Color:</span>
          <span class="detail-value">{{ car.color || 'No especificado' }}</span>
        </div>
        <div class="detail-item">
          <span class="detail-label">Origen:</span>
          <span class="detail-value">{{ car.origen || 'No especificado' }}</span>
        </div>
      </div>

      <div class="car-features">
        <span class="feature-tag" :style="{ backgroundColor: getColorTag(car.color) }">
          {{ car.color || 'N/A' }}
        </span>
        <span class="feature-tag feature-origin">
          {{ car.origen || 'N/A' }}
        </span>
      </div>

      <div class="car-actions">
        <button class="btn btn-primary btn-full" @click="viewDetails">
          <span class="btn-icon">🚗</span>
          Ver detalles
        </button>
        <button class="btn btn-secondary btn-full" @click="contactSeller">
          <span class="btn-icon">💬</span>
          Contactar
        </button>
      </div>
    </div>
  </article>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  car: { 
    type: Object, 
    required: true,
    default: () => ({})
  }
})

const emit = defineEmits(['view-details', 'contact-seller'])

// Imágenes de placeholder por marca
const brandImages = {
  'Toyota': 'https://images.unsplash.com/photo-1580273916550-e323be2ae537?w=400&h=250&fit=crop',
  'Ford': 'https://images.unsplash.com/photo-1549317661-bd32c8ce0db2?w=400&h=250&fit=crop',
  'Honda': 'https://images.unsplash.com/photo-1552519507-da3b142c6e3d?w=400&h=250&fit=crop',
  'Nissan': 'https://images.unsplash.com/photo-1570733577525-d0f20da76c1f?w=400&h=250&fit=crop',
  'Chevrolet': 'https://images.unsplash.com/photo-1621007947382-bb3c3994e3fb?w=400&h=250&fit=crop',
  'Volkswagen': 'https://images.unsplash.com/photo-1603584173870-7f23fdae1b7a?w=400&h=250&fit=crop',
  'BMW': 'https://images.unsplash.com/photo-1555215695-3004980ad54e?w=400&h=250&fit=crop',
  'Mercedes-Benz': 'https://images.unsplash.com/photo-1563720223485-41b7e8b26c4f?w=400&h=250&fit=crop',
  'Audi': 'https://images.unsplash.com/photo-1606664515524-ed2f786a0bd6?w=400&h=250&fit=crop'
}

const imageError = ref(false)

const getCarImage = (car) => {
  if (car.img && !imageError.value) {
    return car.img
  }
  return brandImages[car.marca] || 'https://images.unsplash.com/photo-1544636331-e26879cd4d9b?w=400&h=250&fit=crop'
}

const handleImageError = () => {
  imageError.value = true
}

const formatPrice = (price) => {
  if (!price) return 'Consultar precio'
  return new Intl.NumberFormat('es-MX', { 
    style: 'currency', 
    currency: 'MXN',
    minimumFractionDigits: 0
  }).format(price)
}

const getColorTag = (color) => {
  const colorMap = {
    'rojo': '#FF204E',
    'azul': '#00224D',
    'negro': '#2D3748',
    'blanco': '#E2E8F0',
    'gris': '#718096',
    'plateado': '#CBD5E0',
    'verde': '#38A169',
    'amarillo': '#D69E2E'
  }
  return colorMap[color?.toLowerCase()] || '#6c757d'
}

const viewDetails = () => {
  emit('view-details', props.car)
}

const contactSeller = () => {
  emit('contact-seller', props.car)
}
</script>

<style scoped>
.car-card {
  background: var(--white);
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border: 1px solid #e9ecef;
}

.car-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 25px rgba(0, 0, 0, 0.15);
}

.car-image-container {
  position: relative;
  height: 200px;
  overflow: hidden;
}

.car-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.car-card:hover .car-image {
  transform: scale(1.05);
}

.year-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  background: var(--primary);
  color: var(--white);
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 600;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.image-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 34, 77, 0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.car-card:hover .image-overlay {
  opacity: 1;
}

.quick-view-btn {
  background: var(--white);
  color: var(--dark);
  border: none;
  padding: 10px 20px;
  border-radius: 25px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.quick-view-btn:hover {
  background: var(--primary);
  color: var(--white);
}

.car-content {
  padding: 1.5rem;
}

.car-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
}

.car-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--dark);
  margin: 0;
  line-height: 1.3;
}

.car-price {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--primary);
  text-align: right;
}

.car-details {
  margin-bottom: 1rem;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
}

.detail-label {
  color: var(--gray);
  font-weight: 500;
}

.detail-value {
  color: var(--dark);
  font-weight: 600;
}

.car-features {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
}

.feature-tag {
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--white);
  text-transform: capitalize;
}

.feature-origin {
  background: var(--accent);
}

.car-actions {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.btn-full {
  width: 100%;
  justify-content: center;
}

.btn-icon {
  margin-right: 0.5rem;
}

/* Responsive */
@media (max-width: 768px) {
  .car-header {
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .car-price {
    text-align: left;
  }
  
  .car-actions {
    flex-direction: column;
  }
}
</style>