
<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const dolar = ref([])
const cargando = ref(true)
const errorApi = ref('')
const tipo_dolar = ref('blue')
const opcion = ref('comprar')
const monto = ref('')
const resultado = ref(null)

let intervalo = null

const nombres = {
  oficial: 'Dólar Oficial',
  blue: 'Dólar Blue',
  bolsa: 'Dólar Bolsa',
  contadoconliqui: 'Dólar CCL',
  cripto: 'Dólar Cripto',
  tarjeta: 'Dólar Tarjeta',
  mayorista: 'Dólar Mayorista'
}

const orden = [
  'oficial',
  'blue',
  'bolsa',
  'contadoconliqui',
  'cripto',
  'tarjeta',
  'mayorista'
]

const dolaresOrdenados = computed(() => {
  return [...dolar.value].sort((a, b) => {
    const posicionA = orden.indexOf(a.casa)
    const posicionB = orden.indexOf(b.casa)

    return (posicionA < 0 ? 999 : posicionA) -
           (posicionB < 0 ? 999 : posicionB)
  })
})

const dolarSeleccionado = computed(() => {
  return dolar.value.find(d => d.casa === tipo_dolar.value)
})

const formatoARS = valor => {
  return new Intl.NumberFormat('es-AR', {
    style: 'currency',
    currency: 'ARS',
    maximumFractionDigits: 2
  }).format(valor)
}

const formatoUSD = valor => {
  return new Intl.NumberFormat('es-AR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(valor) + ' USD'
}

const fechaActualizacion = computed(() => {
  const fecha = dolar.value.find(d => d.fechaActualizacion)
    ?.fechaActualizacion

  if (!fecha) return ''

  return new Date(fecha).toLocaleString('es-AR')
})

async function mostrar() {
  try {
    errorApi.value = ''

    const respuesta = await fetch(
      'https://dolarapi.com/v1/dolares'
    )

    if (!respuesta.ok) {
      throw new Error('No se pudieron obtener las cotizaciones.')
    }

    const datos = await respuesta.json()

    if (!Array.isArray(datos)) {
      throw new Error('La API devolvió un formato inesperado.')
    }

    dolar.value = datos
  } catch (error) {
    errorApi.value = error.message
  } finally {
    cargando.value = false
  }
}

function operacion() {
  const cantidad = Number(monto.value)
  const cotizacion = dolarSeleccionado.value

  if (!Number.isFinite(cantidad) || cantidad <= 0) {
    resultado.value = {
      error: 'Ingresá un monto mayor que cero.'
    }
    return
  }

  if (!cotizacion) {
    resultado.value = {
      error: 'No se encontró la cotización seleccionada.'
    }
    return
  }

  const precio = opcion.value === 'comprar'
    ? Number(cotizacion.venta)
    : Number(cotizacion.compra)

  if (!Number.isFinite(precio) || precio <= 0) {
    resultado.value = {
      error: 'La cotización no está disponible.'
    }
    return
  }

  if (opcion.value === 'comprar') {
    // El monto ingresado está expresado en pesos.
    resultado.value = {
      error: '',
      titulo: 'Dólares que podés comprar',
      total: formatoUSD(cantidad / precio),
      detalle: `${formatoARS(cantidad)} ÷ ${formatoARS(precio)} por dólar`,
      tipo: nombres[cotizacion.casa] || cotizacion.nombre
    }
  } else {
    // El monto ingresado está expresado en dólares.
    resultado.value = {
      error: '',
      titulo: 'Pesos que recibirías',
      total: formatoARS(cantidad * precio),
      detalle: `${formatoUSD(cantidad)} × ${formatoARS(precio)} por dólar`,
      tipo: nombres[cotizacion.casa] || cotizacion.nombre
    }
  }
}

onMounted(() => {
  mostrar()
  intervalo = setInterval(mostrar, 5 * 60 * 1000)
})

onBeforeUnmount(() => {
  if (intervalo) clearInterval(intervalo)
})
</script>

<template>
  <main class="app-container">

    <header class="header">
      <a href="#" class="brand" aria-label="DolarApp, inicio">
        <span class="brand-icon">$</span>
        <span class="brand-name">Dolar<span>App</span></span>
      </a>

      <span class="header-status">
        <span class="status-dot"></span>
        Cotizaciones
      </span>
    </header>

    <section class="hero">
      <p class="eyebrow">MERCADO CAMBIARIO ARGENTINO</p>
      <h1>Cotización del <span>dólar</span></h1>
      <p class="hero-description">
        Consultá las cotizaciones y calculá tus operaciones.
      </p>
    </section>

    <section class="quotes-section">
      <div class="section-heading">
        <div>
          <h2>Cotizaciones actuales</h2>
          <p>Precios de compra y venta en pesos argentinos.</p>
        </div>
        <button class="refresh-button" @click="mostrar">
          ↻ Actualizar
        </button>
      </div>

      <p v-if="cargando" class="message">
        Cargando cotizaciones...
      </p>

      <div v-else-if="errorApi" class="message error-message">
        {{ errorApi }}
        <button class="refresh-button" @click="mostrar">
          Reintentar
        </button>
      </div>

      <div v-else class="quotes-grid">
        <article
          v-for="item in dolaresOrdenados"
          :key="item.casa"
          class="quote-card"
        >
          <div class="quote-heading">
            <span class="quote-icon">$</span>
            <h3>{{ nombres[item.casa] || item.nombre }}</h3>
          </div>

          <div class="quote-prices">
            <div class="price-box buy">
              <span>Compra</span>
              <strong>{{ formatoARS(item.compra) }}</strong>
            </div>

            <div class="price-box sell">
              <span>Venta</span>
              <strong>{{ formatoARS(item.venta) }}</strong>
            </div>
          </div>
        </article>
      </div>

      <p v-if="fechaActualizacion" class="update-info">
        Última actualización informada por la API:
        {{ fechaActualizacion }}
      </p>
    </section>

    <section class="operation-section">
      <div class="operation-info">
        <span class="operation-icon">↔</span>
        <p class="eyebrow">SIMULADOR</p>
        <h2>Compra y venta</h2>
        <p>
          Elegí el tipo de dólar, la operación y el monto
          para calcular un resultado estimado.
        </p>

        <div class="helper-card">
          <strong>¿Cómo funciona?</strong>
          <p>
            Comprar: ingresá los pesos que querés invertir.
            Vender: ingresá la cantidad de dólares que tenés.
          </p>
        </div>
      </div>

      <form class="operation-form" @submit.prevent="operacion">
        <div class="form-group">
          <label for="tipoOperacion">Tipo de operación</label>
          <select id="tipoOperacion" v-model="opcion">
            <option value="comprar">Comprar dólares</option>
            <option value="vender">Vender dólares</option>
          </select>
        </div>

        <div class="form-group">
          <label for="tipoDolar">Tipo de dólar</label>
          <select id="tipoDolar" v-model="tipo_dolar">
            <option
              v-for="item in dolaresOrdenados"
              :key="item.casa"
              :value="item.casa"
            >
              {{ nombres[item.casa] || item.nombre }}
            </option>
          </select>
        </div>

        <div class="form-group">
          <label for="monto">
            {{ opcion === 'comprar'
              ? 'Monto en pesos argentinos'
              : 'Cantidad de dólares a vender' }}
          </label>

          <div class="input-wrapper">
            <span>{{ opcion === 'comprar' ? '$' : 'USD' }}</span>
            <input
              id="monto"
              v-model="monto"
              type="number"
              min="0.01"
              step="any"
              placeholder="Ingresá un monto"
              required
            />
          </div>
        </div>

        <button class="submit-button" type="submit">
          Calcular operación
          <span>→</span>
        </button>
      </form>

      <div
        class="operation-result"
        :class="{ 'result-error': resultado?.error }"
        aria-live="polite"
      >
        <template v-if="resultado && !resultado.error">
          <span class="result-label">RESULTADO ESTIMADO</span>
          <h3>{{ resultado.titulo }}</h3>
          <strong class="result-total">{{ resultado.total }}</strong>
          <p>{{ resultado.tipo }}</p>
          <p class="result-detail">{{ resultado.detalle }}</p>
        </template>

        <template v-else-if="resultado?.error">
          <span class="result-label">ATENCIÓN</span>
          <p>{{ resultado.error }}</p>
        </template>

        <template v-else>
          <span class="result-label">RESULTADO ESTIMADO</span>
          <h3>Tu operación aparecerá acá</h3>
          <p>
            Completá los campos y presioná
            “Calcular operación”.
          </p>
        </template>
      </div>
    </section>

    <footer class="footer">
      <span>DolarApp</span>
      <p>
        Datos de
        <a
          href="https://dolarapi.com/"
          target="_blank"
          rel="noopener noreferrer"
        >DolarApi.com</a>
        · Simulación orientativa, no ejecuta operaciones reales.
      </p>
    </footer>

  </main>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

:root {
  font-family: 'Inter', Arial, sans-serif;
  color: #f1f5f9;
  background: #0b1120;

  --background: #0b1120;
  --surface: #151e30;
  --border: #29364b;
  --text: #f1f5f9;
  --muted: #94a3b8;
  --primary: #818cf8;
  --green: #34d399;
  --red: #fb7185;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  background:
    radial-gradient(ellipse at top, #172440 0%, #0b1120 55%);
}

button,
input,
select {
  font: inherit;
}

button,
select {
  cursor: pointer;
}

button:focus-visible,
input:focus-visible,
select:focus-visible,
a:focus-visible {
  outline: 2px solid var(--primary);
  outline-offset: 3px;
}

#app {
  min-height: 100vh;
}

.app-container {
  width: min(1120px, 100%);
  margin: 0 auto;
  padding: 25px 28px;
}

/* Encabezado */

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  padding-bottom: 22px;
  border-bottom: 1px solid var(--border);
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  color: var(--text);
  text-decoration: none;
}

.brand-icon {
  width: 38px;
  height: 38px;
  display: grid;
  place-items: center;
  background: linear-gradient(135deg, #818cf8, #6366f1);
  border-radius: 11px;
  color: white;
  font-size: 23px;
  font-weight: 800;
}

.brand-name {
  font-size: 20px;
  font-weight: 800;
}

.brand-name span {
  color: var(--primary);
}

.header-status {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 9px 13px;
  border: 1px solid #24514a;
  background: #142b2a;
  border-radius: 30px;
  color: #a7f3d0;
  font-size: 12px;
  font-weight: 600;
}

.status-dot {
  width: 7px;
  height: 7px;
  background: var(--green);
  border-radius: 50%;
}

/* Presentación */

.hero {
  padding: 32px 0 29px;
}

.eyebrow {
  color: var(--primary);
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 1.6px;
}

.hero h1 {
  margin-top: 10px;
  font-size: clamp(28px, 4vw, 38px);
  line-height: 1.2;
  letter-spacing: -1.3px;
}

.hero h1 span {
  color: var(--primary);
}

.hero-description {
  margin-top: 10px;
  color: var(--muted);
  font-size: 14px;
  line-height: 1.7;
}

/* Cotizaciones */

.quotes-section {
  margin-bottom: 34px;
}

.section-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 14px;
  margin-bottom: 17px;
}

.section-heading h2,
.operation-info h2 {
  font-size: 21px;
  letter-spacing: -0.5px;
}

.section-heading p {
  margin-top: 6px;
  color: var(--muted);
  font-size: 12px;
}

.refresh-button {
  padding: 10px 13px;
  color: #c7d2fe;
  background: #202b45;
  border: 1px solid #394767;
  border-radius: 9px;
  font-size: 12px;
  font-weight: 600;
  transition: background 0.2s;
}

.refresh-button:hover {
  background: #303e60;
}

.quotes-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}

.quote-card {
  min-width: 0;
  padding: 18px;
  background: linear-gradient(145deg, #182338, #131c2d);
  border: 1px solid var(--border);
  border-radius: 13px;
  transition: transform 0.2s, border-color 0.2s;
}

.quote-card:hover {
  transform: translateY(-3px);
  border-color: #5965a6;
}

.quote-heading {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 17px;
}

.quote-icon {
  width: 34px;
  height: 34px;
  flex-shrink: 0;
  display: grid;
  place-items: center;
  border: 1px solid #39436c;
  border-radius: 9px;
  background: #252d50;
  color: #a5b4fc;
  font-size: 19px;
  font-weight: 700;
}

.quote-heading h3 {
  font-size: 14px;
  font-weight: 700;
}

.quote-prices {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 9px;
}

.price-box {
  min-width: 0;
  padding: 11px 9px;
  display: flex;
  flex-direction: column;
  gap: 7px;
  border-radius: 8px;
}

.price-box span {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.price-box strong {
  font-size: clamp(12px, 1.3vw, 16px);
  overflow-wrap: anywhere;
}

.price-box.buy {
  background: #102c2a;
  border: 1px solid #1d4941;
}

.price-box.buy span {
  color: #6ee7b7;
}

.price-box.buy strong {
  color: var(--green);
}

.price-box.sell {
  background: #301d2b;
  border: 1px solid #503040;
}

.price-box.sell span {
  color: #fda4af;
}

.price-box.sell strong {
  color: var(--red);
}

.message {
  padding: 22px;
  border: 1px solid var(--border);
  background: var(--surface);
  border-radius: 12px;
  color: var(--muted);
  font-size: 14px;
}

.error-message {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
  color: #fda4af;
}

.update-info {
  margin-top: 14px;
  color: var(--muted);
  font-size: 11px;
  line-height: 1.6;
}

/* Simulador */

.operation-section {
  display: grid;
  grid-template-columns: 0.8fr 1.2fr;
  gap: 24px;
  padding: 25px;
  background: linear-gradient(135deg, #172239, #121b2b);
  border: 1px solid var(--border);
  border-radius: 16px;
}

.operation-info {
  padding: 4px 0;
}

.operation-icon {
  width: 43px;
  height: 43px;
  display: grid;
  place-items: center;
  margin-bottom: 20px;
  border-radius: 12px;
  background: #282b52;
  color: #c7d2fe;
  font-size: 25px;
}

.operation-info h2 {
  margin-top: 10px;
  font-size: 25px;
}

.operation-info > p:last-child {
  margin-top: 11px;
  color: var(--muted);
  font-size: 13px;
  line-height: 1.8;
}

.helper-card {
  margin-top: 22px;
  padding: 14px;
  background: #10192a;
  border: 1px solid var(--border);
  border-radius: 10px;
}

.helper-card strong {
  font-size: 12px;
  color: #c7d2fe;
}

.helper-card p {
  margin-top: 7px;
  color: var(--muted);
  font-size: 11px;
  line-height: 1.7;
}

/* Formulario */

.operation-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  color: #dbe4f0;
  font-size: 12px;
  font-weight: 600;
}

.form-group select,
.form-group input {
  width: 100%;
  min-height: 45px;
  padding: 0 12px;
  background: #0d1525;
  border: 1px solid #344158;
  border-radius: 9px;
  color: var(--text);
  font-size: 13px;
}

.form-group select:focus,
.form-group input:focus {
  border-color: var(--primary);
}

.form-group select option {
  background: #151e30;
  color: white;
}

.input-wrapper {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-left: 12px;
  background: #0d1525;
  border: 1px solid #344158;
  border-radius: 9px;
}

.input-wrapper:focus-within {
  border-color: var(--primary);
}

.input-wrapper > span {
  color: var(--green);
  font-size: 14px;
  font-weight: 700;
}

.input-wrapper input {
  border: none;
  background: transparent;
}

.submit-button {
  min-height: 47px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 12px 17px;
  background: linear-gradient(135deg, #818cf8, #6366f1);
  border: none;
  border-radius: 9px;
  color: white;
  font-size: 13px;
  font-weight: 700;
  box-shadow: 0 5px 16px rgb(99 102 241 / 20%);
  transition: transform 0.2s, filter 0.2s;
}

.submit-button:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.submit-button span {
  font-size: 18px;
}

/* Resultado */

.operation-result {
  grid-column: 1 / -1;
  display: flex;
  flex-direction: column;
  gap: 9px;
  padding: 19px;
  background: #102c2a;
  border: 1px solid #24514a;
  border-radius: 11px;
}

.result-label {
  color: #a7f3d0;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 1px;
}

.operation-result h3 {
  font-size: 14px;
}

.result-total {
  color: var(--green);
  font-size: clamp(22px, 3vw, 29px);
  overflow-wrap: anywhere;
}

.operation-result p {
  color: #b4ccc6;
  font-size: 12px;
  line-height: 1.6;
}

.operation-result .result-detail {
  color: #89aaa1;
  overflow-wrap: anywhere;
}

.operation-result.result-error {
  background: #301d2b;
  border-color: #503040;
}

.result-error .result-label,
.result-error p {
  color: #fda4af;
}

/* Pie */

.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-top: 24px;
  padding: 18px 0 8px;
  border-top: 1px solid var(--border);
}

.footer > span {
  color: #c7d2fe;
  font-size: 13px;
  font-weight: 700;
}

.footer p {
  color: var(--muted);
  font-size: 11px;
  line-height: 1.7;
}

.footer a {
  color: #a5b4fc;
  text-decoration: none;
}

.footer a:hover {
  text-decoration: underline;
}

/* Tablet */

@media (max-width: 800px) {
  .app-container {
    padding: 22px 18px;
  }

  .quotes-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .operation-section {
    grid-template-columns: 1fr;
    gap: 24px;
    padding: 21px;
  }

  .operation-result {
    grid-column: auto;
  }

  .helper-card {
    margin-top: 15px;
  }
}

/* Celulares */

@media (max-width: 480px) {
  .app-container {
    padding: 16px 12px;
  }

  .header {
    padding-bottom: 17px;
  }

  .brand-icon {
    width: 33px;
    height: 33px;
  }

  .brand-name {
    font-size: 18px;
  }

  .header-status {
    padding: 8px 9px;
    font-size: 10px;
  }

  .hero {
    padding: 27px 0 24px;
  }

  .hero h1 {
    font-size: 29px;
  }

  .hero-description {
    font-size: 13px;
  }

  .section-heading h2 {
    font-size: 18px;
  }

  .section-heading p {
    max-width: 230px;
    line-height: 1.6;
  }

  .refresh-button {
    padding: 9px;
    font-size: 11px;
  }

  .quotes-grid {
    grid-template-columns: 1fr;
    gap: 11px;
  }

  .quote-card {
    padding: 16px;
  }

  .quote-heading h3 {
    font-size: 15px;
  }

  .price-box strong {
    font-size: 15px;
  }

  .operation-section {
    padding: 17px;
  }

  .operation-info h2 {
    font-size: 23px;
  }

  .footer {
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    transition: none !important;
  }
}
</style>
