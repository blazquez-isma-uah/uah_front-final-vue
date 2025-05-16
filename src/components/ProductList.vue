<template>
    <div class="row">
      <!-- Iteramos sobre las categorías -->
      <div
        v-for="category in categories"
        :key="category.id"
        class="row category-container mb-5"
      >
        <!-- Título de la categoría con botón de colapsar/expandir -->
        <h3
          class="category-title text-center bg-light p-2 rounded border mb-3"
          @click="toggleCategory(category.id)"
        >
          {{ category.name }}
        </h3>
  
        <!-- Contenedor con productos filtrados por categoría -->
        <div
          class="row category-container"
          :class="{ expanded: isOpen(category.id), collapsed: !isOpen(category.id) }"
          :id="'category-' + category.id"
        >
          <ProductCard
            v-for="product in productsByCategory(category.id)"
            :key="product.id"
            :product="product"
            @add-to-cart="emitAddToCart"
          />
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { reactive } from 'vue'
  import ProductCard from './ProductCard.vue'
  
  // Recibimos props
  const props = defineProps({
    categories: Array,
    products: Array
  })
  
  const emit = defineEmits(['add-to-cart'])
  
  // Estado reactivo para controlar qué categorías están abiertas
  const openCategories = reactive({})
  
  // Inicializa todas como abiertas
  props.categories.forEach(cat => {
    openCategories[cat.id] = true
  })
  
  // Cambia el estado abierto/cerrado de una categoría
  const toggleCategory = (id) => {
    openCategories[id] = !openCategories[id]
  }
  
  // Devuelve si una categoría está abierta
  const isOpen = (id) => openCategories[id]
  
  // Filtra productos por categoría
  const productsByCategory = (categoryId) =>
    props.products.filter(p => p.categoryId === categoryId)
  
  // Emite evento al padre cuando se agrega un producto
  const emitAddToCart = (productId) => {
    emit('add-to-cart', productId)
  }
  </script>
  