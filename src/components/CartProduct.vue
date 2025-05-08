<template>
    <li class="list-group-item d-flex align-items-center">
      <!-- Imagen del producto -->
      <img
        :src="product.images[0]"
        :alt="item.name"
        class="me-2"
        style="width: 50px; height: 50px; object-fit: cover;"
      />
  
      <!-- Nombre y subtotal -->
      <span>
        <strong>{{ item.name }}</strong> (x{{ item.quantity }}) – ${{ item.price * item.quantity }}
      </span>
  
      <!-- Botón para eliminar una unidad -->
      <button
        class="btn btn-sm btn-danger ms-auto"
        @click="$emit('remove', item.id)"
      >
        Eliminar
      </button>
    </li>
  </template>
  
  <script setup>
  // Recibimos los datos del ítem y su producto completo
  const props = defineProps({
    item: Object,
    product: Object
  })
  </script>
  