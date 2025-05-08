<template>
    <div class="card p-4 mt-4">
      <!-- Vista para introducir la contraseña -->
      <template v-if="!isAdmin">
        <h4>Acceso de Administrador</h4>
        <input
          type="password"
          class="form-control mb-3"
          v-model="password"
          placeholder="Introduce la contraseña"
        />
        <button class="btn btn-primary" @click="handleLogin">Entrar</button>
      </template>
  
      <!-- Vista del panel una vez autenticado -->
      <template v-else>
        <h4>Panel de Administración</h4>
  
        <!-- Botones de utilidad -->
        <div class="d-flex justify-content-center align-items-center px-4 m-3">
          <button class="btn btn-danger me-3" @click="handleResetStore">Resetear Tienda</button>
          <button class="btn btn-success me-3" @click="handleExportStore">Exportar Tienda</button>
          <div class="custom-file-upload">
            <label for="fileInput" class="btn btn-primary">Importar Tienda (.json)</label>
            <input
              id="fileInput"
              type="file"
              class="d-none"
              @change="handleImportStore"
              accept=".json"
            />
          </div>
        </div>
  
        <!-- Añadir categoría -->
        <div class="m-4">
          <h5>Añadir Categoría</h5>
          <input
            type="text"
            class="form-control mb-2"
            v-model="newCategory"
            placeholder="Nombre de la categoría"
          />
          <button class="btn btn-secondary" @click="handleAddCategory">Agregar Categoría</button>
        </div>
  
        <!-- Añadir producto -->
        <div class="m-4">
          <h5>Añadir Producto</h5>
          <input class="form-control mb-2" placeholder="Nombre" v-model="newProduct.name" />
          <input class="form-control mb-2" placeholder="Código" v-model="newProduct.code" />
          <input class="form-control mb-2" placeholder="Descripción" v-model="newProduct.description" />
          <input type="number" class="form-control mb-2" placeholder="Precio" v-model="newProduct.price" />
          <input type="number" class="form-control mb-2" placeholder="Stock" v-model="newProduct.stock" />
  
          <select class="form-select mb-2" v-model="newProduct.categoryId">
            <option value="">Seleccione categoría</option>
            <option v-for="cat in categories" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
          </select>
  
          <input type="file" class="form-control mb-2" multiple @change="handleImageUpload" />
  
          <button class="btn btn-secondary" @click="handleAddProduct">Agregar Producto</button>
        </div>
      </template>
    </div>
  </template>
  
  <script setup>
  // Importar funciones reactivas
  import { ref } from 'vue'
  import defaultData from '../data/data.json'
  
  // Definir props y métodos que el padre pasa
  const props = defineProps({
    categories: Array,
    setCategories: Function,
    products: Array,
    setProducts: Function
  })
  
  // Estado del formulario
  const isAdmin = ref(false)
  const password = ref('')
  const newCategory = ref('')
  
  // Estructura base para nuevo producto
  const newProduct = ref({
    name: '',
    code: '',
    description: '',
    price: '',
    stock: '',
    categoryId: '',
    images: []
  })
  
  // Validación de contraseña
  const handleLogin = () => {
    if (password.value === '123456') {
      isAdmin.value = true
    } else {
      alert('Contraseña incorrecta')
    }
  }
  
  // Añadir una nueva categoría
  const handleAddCategory = () => {
    if (!newCategory.value.trim()) return
  
    const exists = props.categories.some(
      c => c.name.toLowerCase() === newCategory.value.trim().toLowerCase()
    )
    if (exists) {
      alert('Categoría ya existe')
      return
    }
  
    const newId = Math.max(...props.categories.map(c => c.id), 0) + 1
    const updated = [...props.categories, { id: newId, name: newCategory.value.trim() }]
    props.setCategories(updated)
    newCategory.value = ''
  }
  
  // Añadir un nuevo producto
  const handleAddProduct = () => {
    const p = newProduct.value
    if (!p.name || !p.code || !p.description || !p.price || !p.stock || !p.categoryId || p.images.length === 0) {
      alert('Todos los campos son obligatorios')
      return
    }
  
    const exists = props.products.some(pr => pr.id === parseInt(p.code))
    if (exists) {
      alert('Código de producto duplicado')
      return
    }
  
    const prod = {
      id: parseInt(p.code),
      name: p.name,
      description: p.description,
      price: parseFloat(p.price),
      stock: parseInt(p.stock),
      categoryId: parseInt(p.categoryId),
      images: [...p.images]
    }
  
    const updated = [...props.products, prod]
    props.setProducts(updated)
  
    newProduct.value = {
      name: '',
      code: '',
      description: '',
      price: '',
      stock: '',
      categoryId: '',
      images: []
    }
  
    alert('Producto agregado')
  }
  
  // Asignar imágenes al producto (nombre de archivo desde /public/img/)
  const handleImageUpload = (e) => {
    const files = Array.from(e.target.files)
    const paths = files.map(f => `/img/${f.name}`)
    newProduct.value.images = paths
  }
  
  // Restaurar tienda a los datos originales
  const handleResetStore = () => {
    if (!confirm('¿Estás seguro de que quieres resetear la tienda?')) return
  
    const cleanProducts = defaultData.products.map(p => ({
      ...p,
      stock: Number(p.stock)
    }))
  
    localStorage.setItem('products', JSON.stringify(cleanProducts))
    localStorage.setItem('categories', JSON.stringify(defaultData.categories))
  
    props.setProducts(cleanProducts)
    props.setCategories(defaultData.categories)
  
    alert('Tienda reseteada')
  }
  
  // Exportar tienda como archivo JSON
  const handleExportStore = () => {
    const content = {
      categories: props.categories,
      products: props.products
    }
  
    const blob = new Blob([JSON.stringify(content, null, 2)], { type: 'application/json' })
    const url = URL.createObjectURL(blob)
  
    const link = document.createElement('a')
    link.href = url
    link.download = 'tienda_exportada.json'
    link.click()
  
    URL.revokeObjectURL(url)
  }
  
  // Importar tienda desde archivo JSON
  const handleImportStore = (e) => {
    const file = e.target.files[0]
    if (!file) return
  
    const reader = new FileReader()
    reader.onload = (event) => {
      try {
        const imported = JSON.parse(event.target.result)
  
        if (!imported.categories || !imported.products) {
          alert('Estructura inválida')
          return
        }
  
        localStorage.setItem('categories', JSON.stringify(imported.categories))
        localStorage.setItem('products', JSON.stringify(imported.products))
  
        props.setCategories(imported.categories)
        props.setProducts(imported.products)
  
        alert('Importación exitosa')
      } catch (err) {
        alert('Error al leer el archivo JSON')
      }
    }
  
    reader.readAsText(file)
  }
  </script>
  