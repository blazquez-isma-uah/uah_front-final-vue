<template>
    <div id="cart-section">
      <h2>Cesta de la Compra</h2>
  
      <!-- Lista de productos en el carrito -->
      <ul id="cart-list" class="list-group">
        <CartProduct
          v-for="item in cart"
          :key="item.id"
          :item="item"
          :product="findProduct(item.id)"
          @remove="emitRemoveFromCart"
        />
      </ul>
  
      <!-- Total general -->
      <div class="mt-3">
        <strong>Total: <span id="cart-total"> ${{ total }}</span></strong>
      </div>
  
      <!-- Botón para realizar pedido -->
      <button
        id="checkout-button"
        class="btn btn-success mt-3"
        @click="$emit('checkout')"
      >
        Realizar Pedido
      </button>
    </div>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  import CartProduct from './CartProduct.vue'
  
  // Recibimos props
  const props = defineProps({
    cart: Array,
    products: Array
  })
  
  const emit = defineEmits(['remove-from-cart', 'checkout'])
  
  // Busca un producto completo por su ID
  const findProduct = (id) => {
    return props.products.find(p => p.id === id)
  }
  
  // Calcula el total del carrito
  const total = computed(() =>
    props.cart.reduce((sum, item) => sum + item.price * item.quantity, 0).toFixed(2)
  )
  
  // Emitir evento al hacer clic en eliminar
  const emitRemoveFromCart = (productId) => {
    emit('remove-from-cart', productId)
  }
  </script>
  