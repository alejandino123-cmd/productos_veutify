<template>
  <div class="contenedor-login">
    <div class="tarjeta-login">
      <div class="encabezado-login">
        <h2>Iniciar sesión</h2>
        <p>Ingresa tus credenciales para acceder al sistema</p>
      </div>

      <form @submit.prevent="enviarLogin" class="formulario-login">
        <div class="campo-formulario">
          <v-text-field
            id="email"
            v-model="credenciales.email"
            label="Correo electrónico"
            type="email"
            placeholder="tu@email.com"
            autocomplete="username"
            variant="outlined"
            :error-messages="errores.email"
            class="input-completo"
          />
        </div>

        <div class="campo-formulario">
          <v-text-field
            id="password"
            v-model="credenciales.password"
            label="Contraseña"
            placeholder="Ingresa tu contraseña"
            autocomplete="current-password"
            variant="outlined"
            :type="verPassword ? 'text' : 'password'"
            :append-inner-icon="verPassword ? 'mdi-eye-off' : 'mdi-eye'"
            :error-messages="errores.password"
            class="input-completo"
            @click:append-inner="verPassword = !verPassword"
          />
        </div>

        <v-btn type="submit" color="primary" size="large" block :loading="cargando" class="boton-login">
          Iniciar sesión
        </v-btn>
      </form>

      <v-alert v-if="mensajeError" type="error" variant="tonal" class="mensaje-server">
        {{ mensajeError }}
      </v-alert>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthService } from '@/service/api'

const router = useRouter()
const { login } = useAuthService()

const cargando = ref(false)
const mensajeError = ref('')
const verPassword = ref(false)

const credenciales = reactive({
  email: '',
  password: '',
})

const errores = reactive({
  email: '',
  password: '',
})

const validarFormulario = () => {
  let esValido = true
  errores.email = ''
  errores.password = ''

  if (!credenciales.email.trim()) {
    errores.email = 'El correo electrónico es requerido'
    esValido = false
  } else if (!/\S+@\S+\.\S+/.test(credenciales.email)) {
    errores.email = 'El formato del correo no es válido'
    esValido = false
  }

  if (!credenciales.password) {
    errores.password = 'La contraseña es requerida'
    esValido = false
  } else if (credenciales.password.length < 6) {
    errores.password = 'La contraseña debe tener al menos 6 caracteres'
    esValido = false
  }

  return esValido
}

const enviarLogin = async () => {
  if (!validarFormulario()) return

  cargando.value = true
  mensajeError.value = ''

  try {
    const resultado = await login(credenciales)

    if (!resultado.success) {
      mensajeError.value = resultado.error
      return
    }

    // Superadmin: entra directo. Usuario normal: elige cómo verificar su identidad
    if (resultado.requires2fa) {
      router.push({ name: 'elegir-metodo' })
    } else {
      router.push({ name: 'AdminDashboard' })
    }
  } catch {
    mensajeError.value = 'Ocurrió un error al procesar el inicio de sesión'
  } finally {
    cargando.value = false
  }
}
</script>