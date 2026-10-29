<template>
  <div class="contenedor-login">
    <div class="tarjeta-login">
      <div class="encabezado-login">
        <h2>Verifica tu correo</h2>
        <p v-if="correoOculto">
          Enviamos un código de 6 dígitos a <strong>{{ correoOculto }}</strong>. Vence en 10 minutos.
        </p>
        <p v-else>Enviamos un código de 6 dígitos a tu correo electrónico. Vence en 10 minutos.</p>
      </div>

      <!-- Solo aparece en desarrollo (npm run dev); en producción no existe -->
      <v-alert v-if="codigoDebug" type="warning" variant="tonal" class="mensaje-server">
        Modo pruebas. Código generado: <strong>{{ codigoDebug }}</strong>
      </v-alert>

      <form @submit.prevent="enviarCodigo" class="formulario-login">
        <div class="campo-formulario">
          <v-text-field
            id="codigo"
            :model-value="codigo"
            label="Código de verificación"
            inputmode="numeric"
            autocomplete="one-time-code"
            maxlength="6"
            placeholder="000000"
            variant="outlined"
            :error="!!mensajeError"
            class="input-completo input-codigo"
            @update:model-value="soloNumeros"
            @keypress="bloquearNoNumeros"
          />
        </div>

        <v-btn type="submit" color="primary" size="large" block :loading="cargando" class="boton-login">
          Verificar código
        </v-btn>
      </form>

      <v-alert v-if="mensajeError" type="error" variant="tonal" class="mensaje-server">
        {{ mensajeError }}
      </v-alert>

      <div class="pie-verificacion">
        <v-btn variant="text" :disabled="cargando" @click="cambiarMetodo">Usar otro método</v-btn>
        <v-btn variant="text" :disabled="cargando" @click="volverAlLogin">
          Volver a iniciar sesión
        </v-btn>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthService } from '@/service/api'

const router = useRouter()
const { verificarCodigo, cancelarVerificacion } = useAuthService()

const codigo = ref('')
const cargando = ref(false)
const mensajeError = ref('')
const email = ref('')
const codigoDebug = ref('')

// ana.perez@correo.com -> a***@correo.com
const correoOculto = computed(() => {
  const [usuario, dominio] = email.value.split('@')
  return usuario && dominio ? `${usuario[0]}***@${dominio}` : ''
})

onMounted(() => {
  email.value = localStorage.getItem('user_email') || ''

  // import.meta.env.DEV es false en el build de producción
  if (import.meta.env.DEV) {
    codigoDebug.value = localStorage.getItem('debug_code') || ''
  }
})

// Solo deja dígitos (también cuando se pega texto)
const soloNumeros = (valor) => {
  codigo.value = String(valor ?? '').replace(/\D/g, '')
}

// Evita que se escriban letras o símbolos
const bloquearNoNumeros = (evento) => {
  if (!/\d/.test(evento.key)) evento.preventDefault()
}

const enviarCodigo = async () => {
  if (codigo.value.length !== 6) {
    mensajeError.value = 'Escribe los 6 dígitos del código.'
    return
  }

  cargando.value = true
  mensajeError.value = ''

  try {
    const resultado = await verificarCodigo(codigo.value)

    if (!resultado.success) {
      mensajeError.value = resultado.error
      return
    }

    router.push({ name: 'AdminDashboard' })
  } finally {
    cargando.value = false
  }
}

const cambiarMetodo = () => {
  router.push({ name: 'elegir-metodo' })
}

// Para empezar de cero hay que iniciar sesión de nuevo
const volverAlLogin = () => {
  cancelarVerificacion()
  router.push({ name: 'login' })
}
</script>

<style scoped>
:deep(.input-codigo input) {
  text-align: center;
  font-size: 1.5rem;
  letter-spacing: 0.5em;
  font-variant-numeric: tabular-nums;
}

.pie-verificacion {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 0.5rem;
}
</style>