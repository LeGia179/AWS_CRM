<script setup lang="ts">
import { ref, computed } from 'vue'

// Userdaten
const nachName = ref('')
const vorName = ref('')
const email = ref('')
const straße = ref('')
const hausnummer = ref('')
const plz = ref('')

// Produktlogik
const selectItem = ref('Produkte')
const products = ref([{ product: '', quantity: 1 }])

const chooseItem: Record<string, string[]> = {
  Produkte: ['Proteinpulver', 'Kreatin', 'Isoclear']
}

const prices: Record<string, number> = {
  Proteinpulver: 29.99,
  Kreatin: 19.99,
  Isoclear: 24.99
}

const totalPrice = computed(() => {
  return products.value.reduce((sum, item) => {
    const price = prices[item.product] || 0
    return sum + price * item.quantity
  }, 0).toFixed(2)
})

const addProduct = () => {
  products.value.push({ product: '', quantity: 1 })
}

const handlePurchase = () => {
  const summary = products.value
      .filter(p => p.product && p.quantity > 0)
      .map(p => `${p.quantity}x ${p.product}`)
      .join(', ')
  alert(`Bestellung für: ${summary}. Gesamtpreis: €${totalPrice.value}`)

  sendToLambda()
}

const sendToLambda = async () => {
  const payload = {
    nachName: nachName.value,
    vorName: vorName.value,
    email: email.value,
    strasse: straße.value,
    hausnummer: hausnummer.value,
    plz: plz.value,
    produkt: products.value,
    gesamtpreis: totalPrice.value,
  }

  try {
    const response = await fetch('https://l7cvx7hf36.execute-api.eu-west-1.amazonaws.com/produktion/generate-pdf', {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify(payload),
    })

    if (!response.ok) throw new Error('Fehler beim PDF-Generieren')

    const blob = await response.blob()
    const url = URL.createObjectURL(blob)
    window.open(url, '_blank')
  } catch (error) {
    alert('Fehler beim Senden: ' + error)
  }
}


</script>

<template>
  <div class="ui-container">
    <!-- Linker Bereich -->
    <div class="leftside">
      <h3>Kundendaten</h3>
      <label>Nachname</label>
      <input v-model="nachName" type="text" placeholder="Nachname" required />

      <label>Vorname</label>
      <input v-model="vorName" type="text" placeholder="Vorname" required />

      <label>E-Mail</label>
      <input v-model="email" type="email" placeholder="E-Mail" required />

      <label>Straße</label>
      <input v-model="straße" type="text" placeholder="Straße" required />

      <label>Hausnummer</label>
      <input v-model="hausnummer" type="text" placeholder="Hausnummer" required />

      <label>Postleitzahl</label>
      <input v-model="plz" type="text" placeholder="PLZ" required />
    </div>

    <!-- Rechte Seite: Produktauswahl -->
    <div class="rightside">
      <h3>Produktwahl</h3>

      <div
          v-for="(item, index) in products"
          :key="index"
          class="product-line"
      >
        <div class="product-row">
          <select v-model="item.product" class="product-select">
            <option disabled value="">Bitte Produkt wählen</option>
            <option
                v-for="option in chooseItem[selectItem]"
                :key="option"
                :value="option"
            >
              {{ option }}
            </option>
          </select>

          <input
              type="number"
              v-model.number="item.quantity"
              min="1"
              class="product-quantity"
          />

          <span class="product-price" v-if="item.product">
            €{{ (prices[item.product] * item.quantity).toFixed(2) }}
          </span>
        </div>
      </div>

      <button class="add-btn" @click="addProduct">+ Produkt hinzufügen</button>

      <div class="total-section">
        <p><strong>Gesamtpreis: €{{ totalPrice }}</strong></p>
        <button
            @click="handlePurchase"
            :disabled="products.length === 0 || products.some(p => !p.product || p.quantity < 1)"
        >
          Jetzt kaufen
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.ui-container {
  display: flex;
  flex-direction: row;
  gap: 30px;
  padding: 20px;
  justify-content: center;
  align-items: flex-start;
  background-color: #eef3f8;
  overflow-x: auto;
}

/* Linke & rechte Box */
.leftside,
.rightside {
  width: 400px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
  border: 2px solid #000;
  border-radius: 10px;
  background-color: #F4A460;
  min-height: 500px;
  color: #000000;
}

/* Rechte Box scrollt */
.rightside {
  max-height: 500px;
  overflow-y: auto;
  justify-content: flex-start;
}

/* Eingabefelder */
.leftside input,
.rightside input {
  background-color: #F0F8FF;
  color: black;
  width: 100%;
}

/* Produktlinie */
.product-line {
  border: 1px solid #aaa;
  padding: 10px;
  border-radius: 10px;
  background-color: #f1f1f1;
  margin-bottom: 10px;
}

/* Produkt & Menge nebeneinander */
.product-row {
  display: flex;
  gap: 10px;
  align-items: center;
}

/* Weißer Hintergrund + schwarze Schrift für Select */
.product-select {
  background-color: #F0F8FF;
  color: #000000;
  border: 1px solid #000000;
  border-radius: 4px;
  padding: 8px;
  width: 100%;
  appearance: none; /* optional für einheitlichen Look */
}

/* Option-Farben */
.product-select option {
  background-color: #F0F8FF;
  color: #000000;
  border: 1px solid #000000;
}


.product-quantity {
  width: 60px;
  padding: 6px;
  text-align: center;
  border: 1px solid #aaa;
  border-radius: 4px;
}

.product-price {
  font-size: 0.9rem;
  white-space: nowrap;
}

/* Buttons */
button {
  margin-top: 10px;
  padding: 10px;
  font-size: 1rem;
  background-color: #F0F8FF;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  color: #000000;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.add-btn {
  background-color: #F0F8FF;
  color: black;
}

/* Gesamtsumme + Kaufen */
.total-section {
  margin-top: 20px;
  padding-top: 10px;
  border-top: 1px solid #ccc;
}
</style>
