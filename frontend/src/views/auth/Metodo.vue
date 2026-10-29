<template>
  <div class="contenedor-login">
    <div class="tarjeta-login">
      <div class="encabezado-login">
        <h2>Verifica tu identidad</h2>
        <p>Elige cómo quieres confirmar que eres tú</p>
      </div>

      <div class="opciones-metodo">
        <v-btn
          color="primary"
          size="large"
          block
          prepend-icon="mdi-email-outline"
          :loading="enviando"
          class="boton-login"
          @click="porCorreo"
        >
          Recibir código por correo
        </v-btn>
        <v-btn
          color="secondary"
          variant="tonal"
          size="large"
          block
          prepend-icon="mdi-qrcode"
          :disabled="enviando"
          class="boton-login"
          @click="porQr"
        >
          Usar Google Authenticator (QR)
        </v-btn>
      </div>

      <v-alert v-if="mensajeError" type="error" variant="tonal" class="mensaje-server">
        {{ mensajeError }}
      </v-alert>

      <div class="pie-verificacion">
        <v-btn variant="text" :disabled="enviando" @click="volverAlLogin">
          Volver a iniciar sesión
        </v-btn>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthService } from '@/service/api'

const router = useRouter()
const { enviarCodigoCorreo, cancelarVerificacion } = useAuthService()

const enviando = ref(false)
const mensajeError = ref('')

const porCorreo = async () => {
  enviando.value = true
  mensajeError.value = ''

  try {
    const resultado = await enviarCodigoCorreo()

    if (!resultado.success) {
      mensajeError.value = resultado.error
      return
    }

    router.push({ name: 'verificar-codigo' })
  } finally {
    enviando.value = false
  }
}

const porQr = () => {
  router.push({ name: 'two-factor' })
}

const volverAlLogin = () => {
  cancelarVerificacion()
  router.push({ name: 'login' })
}
</script>

<style scoped>
.opciones-metodo {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.pie-verificacion {
  display: flex;
  justify-content: center;
  margin-top: 0.5rem;
}
</style>