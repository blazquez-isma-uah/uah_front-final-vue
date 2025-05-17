<template>
  <div class="App">
    <!-- Cabecera con título y botón para activar modo administrador -->
    <div class="d-flex justify-content-between align-items-center px-4 mt-3">
      <h1 class="m-0">Tienda Online</h1>
      <button class="btn btn-outline-secondary" @click="showAdminPanel = !showAdminPanel">
        <i :class="showAdminPanel ? 'bi bi-x-circle' : 'bi bi-gear'"></i> Admin
      </button>
    </div>

    <div class="container mt-4">
      <!-- Vista del panel de administración si está activo -->
      <AdminPanel v-if="showAdminPanel"
        :products="products"
        :set-products="setProducts"
        :categories="categories"
        :set-categories="setCategories"
      />

      <!-- Vista principal de tienda si no está en modo administrador -->
      <template v-else>
        <div class="row">
          <!-- Lista de productos -->
          <div class="col-md-8">
            <ProductList
              :categories="categories"
              :products="products"
              @add-to-cart="handleAddToCart"
            />
          </div>

          <!-- Cesta de la compra -->
          <div class="col-md-4">
            <Cart
              :cart="cart"
              :products="products"
              @remove-from-cart="handleRemoveFromCart"
              @checkout="handleCheckout"
            />
            <!-- Notificación cuando se añade un producto -->
            <div v-if="showNotification" class="alert alert-success notification">
              <p>Producto agregado al carrito</p>
            </div>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import './assets/styles.css';
import { ref, onMounted } from 'vue'
import data from './data/data.json'

// Importar componentes
import ProductList from './components/ProductList.vue'
import Cart from './components/Cart.vue'
import AdminPanel from './components/AdminPanel.vue'

const products = ref([])
const categories = ref([])
const cart = ref([])

const showNotification = ref(false)
const showAdminPanel = ref(false)

// Cargar datos iniciales
onMounted(() => {
  const storedProducts = localStorage.getItem('products')
  const storedCategories = localStorage.getItem('categories')

  // Cargar categorías desde localStorage o desde JSON
  if (storedCategories) {
    categories.value = JSON.parse(storedCategories)
  } else {
    categories.value = data.categories
    localStorage.setItem('categories', JSON.stringify(data.categories))
  }

  // Cargar productos desde localStorage o desde JSON
  if (storedProducts) {
    products.value = JSON.parse(storedProducts)
  } else {
    const cloned = data.products.map(p => ({ ...p, stock: Number(p.stock) }))
    products.value = cloned
    localStorage.setItem('products', JSON.stringify(cloned))
  }
})

// Actualizar productos y guardar en localStorage
const setProducts = (newList) => {
  products.value = newList
  localStorage.setItem('products', JSON.stringify(newList))
}
// Actualizar categorías y guardar en localStorage
const setCategories = (newList) => {
  categories.value = newList
  localStorage.setItem('categories', JSON.stringify(newList))
}

// Añadir producto al carrito
const handleAddToCart = (productId) => {
  const updated = [...products.value]
  const prod = updated.find(p => p.id === productId)
  if (!prod || prod.stock <= 0) return

  prod.stock--

  const existing = cart.value.find(i => i.id === productId)
  if (existing) {
    existing.quantity++
  } else {
    cart.value.push({
      id: prod.id,
      name: prod.name,
      price: prod.price,
      quantity: 1
    })
  }

  showNotification.value = true
  setTimeout(() => {
    showNotification.value = false
  }, 1000)

  setProducts(updated)
}

// Eliminar producto del carrito
const handleRemoveFromCart = (productId) => {
  const updated = [...cart.value]
  const prod = products.value.find(p => p.id === productId)

  const index = updated.findIndex(i => i.id === productId)
  if (index !== -1 && prod) {
    const item = updated[index]
    item.quantity--
    prod.stock++

    if (item.quantity === 0) updated.splice(index, 1)

    cart.value = updated
    setProducts([...products.value])
  }
}

// Finalizar compra
const handleCheckout = () => {
  if (cart.value.length === 0) {
    alert('El carrito está vacío')
    return
  }

  const total = cart.value.reduce((sum, i) => sum + i.price * i.quantity, 0)
  alert(`Compra realizada. Total: $${total.toFixed(2)}`)
  cart.value = []
}
</script>
