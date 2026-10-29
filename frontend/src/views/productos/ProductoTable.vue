<template>
  <div class="tabla-productos">
    <!-- Botones de exportación -->
    <div class="acciones-tabla mb-4">
      <v-btn color="success" prepend-icon="mdi-file-excel" @click="exportarCSV">
        Exportar CSV
      </v-btn>
      <v-btn color="error" prepend-icon="mdi-file-pdf-box" @click="exportarPDF">
        Exportar PDF
      </v-btn>
    </div>

    <v-card>
      <v-card-title>Lista de Productos</v-card-title>

      <v-card-text>
        <!-- DIV con ID para PDF -->
        <div id="tabla-productos-pdf" class="contenedor-tabla">
          <v-data-table
            v-model:items-per-page="filasPorPagina"
            :headers="columnas"
            :items="productos"
            :loading="cargando"
            :items-per-page-options="[5, 10, 20, 50]"
            items-per-page-text="Filas por página"
            density="comfortable"
          >
            <template #[`item.description`]="{ item }">
              {{ item.description || 'Sin descripción' }}
            </template>

            <template #[`item.price`]="{ item }">
              {{ formatearPrecio(item.price) }}
            </template>

            <template #no-data>
              <div class="estado-vacio">
                <v-icon icon="mdi-inbox" size="40" />
                <p>No se encontraron productos</p>
              </div>
            </template>

            <template #loading>
              <div class="estado-cargando">
                <v-progress-circular indeterminate />
                <p>Cargando productos...</p>
              </div>
            </template>
          </v-data-table>
        </div>
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useProductoService } from '@/service/api'
import jsPDF from 'jspdf'
import autoTable from 'jspdf-autotable'

const { obtenerProductos } = useProductoService()

// Variables reactivas
const productos = ref([])
const cargando = ref(false)
const filasPorPagina = ref(5)

// Columnas de la tabla
const columnas = [
  { title: 'ID', key: 'id', width: '10%' },
  { title: 'Nombre', key: 'name', width: '25%' },
  { title: 'Descripción', key: 'description', width: '30%' },
  { title: 'Precio', key: 'price', width: '15%' },
]

// Métodos de formato
const formatearPrecio = (precio) => {
  return new Intl.NumberFormat('es-MX', {
    style: 'currency',
    currency: 'MXN',
  }).format(precio || 0)
}

// Cargar productos desde la API
const cargarProductos = async () => {
  cargando.value = true
  try {
    const respuesta = await obtenerProductos()
    if (Array.isArray(respuesta)) {
      productos.value = respuesta
    } else if (respuesta && Array.isArray(respuesta.data)) {
      productos.value = respuesta.data
    } else {
      productos.value = []
    }
  } catch (error) {
    console.error('Error al cargar productos:', error)
    productos.value = []
  } finally {
    cargando.value = false
  }
}

// Exportar a PDF
const exportarPDF = () => {
  const doc = new jsPDF()

  // Título del reporte
  doc.setFontSize(18)
  doc.text('Reporte de Inventario - ULS', 14, 20)

  doc.setFontSize(11)
  doc.setTextColor(100)
  doc.text(`Fecha de emisión: ${new Date().toLocaleDateString()}`, 14, 28)

  // Mapear datos para la tabla del PDF
  const cabecera = ['ID', 'Nombre', 'Descripción', 'Precio', 'Stock']
  const filas = productos.value.map((p) => [
    p.id,
    p.name,
    p.description || 'Sin descripción',
    `$${p.price}`,
    p.stock,
  ])

  autoTable(doc, {
    head: [cabecera],
    body: filas,
    startY: 35,
    theme: 'grid',
    headStyles: { fillColor: [41, 128, 185] },
  })

  doc.save('reporte_productos.pdf')
}

// Exportar a CSV
const exportarCSV = () => {
  if (!productos.value.length) return

  const cabeceras = ['ID,Nombre,Descripcion,Precio,Stock']
  const filas = productos.value.map(
    (p) => `"${p.id}","${p.name}","${p.description || ''}","${p.price}","${p.stock}"`,
  )

  const contenidoCSV = [cabeceras, ...filas].join('\n')
  const blob = new Blob([contenidoCSV], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)

  const link = document.createElement('a')
  link.setAttribute('href', url)
  link.setAttribute('download', 'productos_inventario.csv')
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

// Ciclo de vida
onMounted(() => {
  cargarProductos()
})

defineExpose({ cargarProductos })
</script>

<style scoped>
.acciones-tabla {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}
.contenedor-tabla {
  background: white;
  padding: 1rem;
  border-radius: 6px;
}
.estado-vacio,
.estado-cargando {
  padding: 2rem 0;
  text-align: center;
}
</style>