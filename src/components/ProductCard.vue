<template>
    <div class="col-md-4 product-card mb-4">
      <div class="card h-100">
        <!-- Imagen con zoom y cambio de imagen al hacer hover -->
        <div
          class="product-image-wrapper position-relative"
          @mouseenter="startHover"
          @mouseleave="stopHover"
        >
          <div class="zoom-container">
            <img
              :src="currentImage"
              :alt="product.name"
              class="card-img-top product-img"
              :class="{ 'opacity-50': product.stock === 0 }"
            />
          </div>
  
          <!-- Overlay con iconos mientras hace hover -->
          <div v-if="hovering" class="image-overlay">
            <span><i class="bi bi-arrow-repeat" /> <i class="bi bi-camera"></i></span>
          </div>
        </div>
  
        <!-- Información del producto -->
        <div class="card-body d-flex flex-column text-center">
          <h5 class="card-title">{{ product.name }}</h5>
          <p class="text-muted">Código: <strong>{{ product.id }}</strong></p>
          <p class="card-text">{{ product.description }}</p>
          <p class="text-success"><strong>${{ product.price.toFixed(2) }}</strong></p>
          <p class="text-muted">Stock: {{ product.stock }}</p>
  
          <!-- Botón agregar -->
          <button
            class="btn mt-auto"
            :class="product.stock === 0 ? 'btn-secondary' : 'btn-primary'"
            :disabled="product.stock === 0"
            @click="emitAddToCart"
          >
            {{ product.stock === 0 ? 'Agotado' : 'Agregar a la cesta' }}
          </button>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  // Recibimos las props del producto y función del padre
  import { ref, computed, watchEffect, onBeforeUnmount } from 'vue'
  const props = defineProps({
    product: Object
  })
  const emit = defineEmits(['add-to-cart'])
  
  // Estado para hover e imagen actual
  const hovering = ref(false)
  const currentImageIndex = ref(0)
  let interval = null
  let delayTimer = null
  
  // Devuelve la imagen actual según el índice
  const currentImage = computed(() => props.product.images[currentImageIndex.value])
  
  // Al hacer hover, espera 0.5s y luego cambia imagen cada 1.5s
  const startHover = () => {
    hovering.value = true
    delayTimer = setTimeout(() => {
      interval = setInterval(() => {
        currentImageIndex.value =
          (currentImageIndex.value + 1) % props.product.images.length
      }, 1500)
    }, 500)
  }
  
  // Al salir del hover, resetea imagen e intervalos
  const stopHover = () => {
    hovering.value = false
    currentImageIndex.value = 0
    clearTimeout(delayTimer)
    clearInterval(interval)
  }
  
  // Emitimos evento al padre
  const emitAddToCart = () => {
    if (props.product.stock > 0) emit('add-to-cart', props.product.id)
  }
  
  // Limpiar intervalos si el componente se destruye
  onBeforeUnmount(() => {
    clearTimeout(delayTimer)
    clearInterval(interval)
  })
  </script>
  