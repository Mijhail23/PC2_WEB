<template>
  <q-card class="q-pa-xl shadow-2" style="max-width: 420px; margin: auto">
    <div class="text-center q-mb-md">
      <q-icon name="trending_up" color="positive" size="32px" />
      <div class="text-h5 q-mt-sm">Conversión de Monedas</div>
    </div>
    <q-form @submit.prevent="convertir">
      <div class="text-subtitle1 q-mb-xs">Monto a convertir</div>
      <q-input
        v-model.number="monto"
        type="number"
        dense
        outlined
        placeholder="100"
        :error="!!errorMonto"
        :error-message="errorMonto"
        class="q-mb-lg"
      />

      <div class="row q-gutter-md q-mb-md">
        <div class="col">
          <div class="text-caption q-mb-xs">Desde</div>
          <q-select
            v-model="from"
            :options="monedas"
            dense
            outlined
            emit-value
            map-options
            :error="!!errorFrom"
            :error-message="errorFrom"
            option-label="label"
            option-value="value"
            placeholder="Moneda origen"
          />
        </div>
        <div class="col">
          <div class="text-caption q-mb-xs">Hacia</div>
          <q-select
            v-model="to"
            :options="monedas"
            dense
            outlined
            emit-value
            map-options
            :error="!!errorTo"
            :error-message="errorTo"
            option-label="label"
            option-value="value"
            placeholder="Moneda destino"
          />
        </div>
      </div>

      <div class="flex flex-center q-mb-md">
        <q-btn icon="swap_vert" round flat color="primary" @click="intercambiar" size="lg" />
      </div>

      <q-btn
        label="Convertir"
        type="submit"
        color="primary"
        class="full-width"
        style="background: linear-gradient(90deg, #3b82f6 0%, #6366f1 100%)"
      />
    </q-form>

    <q-banner v-if="resultado" class="q-mt-lg bg-grey-2 text-center">
      {{ resultado }}
    </q-banner>
    <q-banner v-if="errorApi" class="q-mt-md bg-red-2 text-red text-center">
      {{ errorApi }}
    </q-banner>
  </q-card>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const monto = ref(null)
const from = ref(null)
const to = ref(null)
const monedas = ref([])
const resultado = ref('')
const errorMonto = ref('')
const errorFrom = ref('')
const errorTo = ref('')
const errorApi = ref('')

onMounted(async () => {
  try {
    const res = await fetch('https://api.frankfurter.app/currencies')
    const data = await res.json()
    monedas.value = Object.entries(data).map(([codigo, nombre]) => ({
      label: `${codigo}  ${nombre}`,
      value: codigo,
    }))
    from.value = 'USD'
    to.value = 'EUR'
  } catch {
    errorApi.value = 'No se pudieron cargar las monedas.'
  }
})

function validar() {
  errorMonto.value = ''
  errorFrom.value = ''
  errorTo.value = ''
  let valido = true

  if (!monto.value || monto.value <= 0) {
    errorMonto.value = 'Ingrese un monto válido'
    valido = false
  }
  if (!from.value) {
    errorFrom.value = 'Seleccione la moneda de origen'
    valido = false
  }
  if (!to.value) {
    errorTo.value = 'Seleccione la moneda de destino'
    valido = false
  }
  return valido
}

async function convertir() {
  resultado.value = ''
  errorApi.value = ''
  if (!validar()) return

  try {
    const res = await fetch(
      `https://api.frankfurter.app/latest?amount=${monto.value}&from=${from.value}&to=${to.value}`,
    )
    const data = await res.json()
    const valor = data.rates[to.value]
    resultado.value = `${monto.value} ${from.value} equivalen a ${valor} ${to.value}`
  } catch {
    errorApi.value = 'Error al obtener la conversión.'
  }
}

function intercambiar() {
  const temp = from.value
  from.value = to.value
  to.value = temp
}
</script>
