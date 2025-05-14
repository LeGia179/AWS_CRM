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
const selectedProduct = ref('')
const quantity = ref(1)

const chooseItem: Record<string, string[]> = {
  Produkte: ['Proteinpulver','Kreatin','Isoclear']
}

const prices: Record<string, number> = {
  Proteinpulver: 29.99,
  Kreatin: 19.99,
  Isoclear: 24.99
}

const totalPrice = computed(() => {
  const price = prices[selectedProduct.value] || 0
  return (price * quantity.value).toFixed(2)
})

const handlePurchase = () => {
  alert(`Bestellung für ${quantity.value}x ${selectedProduct.value} wurde aufgegeben. Gesamtpreis: €${totalPrice.value}`)
}
</script>

<template>
  <div class="ui-container">
    <!-- Linker Bereich: Benutzerdaten -->
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

    <!-- Rechter Bereich: Produktauswahl -->
    <div class="rightside">
      <h3>Produktwahl</h3>
      <label>Produktauswahl:</label>
      <select v-model="selectedProduct">
        <option disabled value="">Bitte Produkt wählen</option>
        <option v-for="option in chooseItem[selectItem]" :key="option" :value="option">
          {{ option }}
        </option>
      </select>

      <label>Menge:</label>
      <input type="number" v-model.number="quantity" min="1" />

      <div v-if="selectedProduct">
        <p>Einzelpreis: €{{ prices[selectedProduct].toFixed(2) }}</p>
        <p><strong>Gesamtpreis: €{{ totalPrice }}</strong></p>
      </div>

      <button @click="handlePurchase" :disabled="!selectedProduct || quantity < 1">Jetzt kaufen</button>
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
  overflow-x: auto; /* Falls Bildschirm zu klein wird */
}

.leftside,
.rightside {
  width: 400px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
  border: 2px solid #aaaaaa;
  border-radius: 10px;
  background-color: #aaaaaa;
  box-shadow: 2px 2px 8px rgba(0, 0, 0, 0.1);
  min-height: 500px; /* Gleiche Höhe für beide Boxen */
}

/* Neue Klasse für den rechten Box-Inhalt */
.rightside {
  justify-content: space-between;
}

h3 {
  margin-bottom: 10px;
}

input,
select {
  padding: 8px;
  border: 1px solid #aaa;
  border-radius: 4px;
}

button {
  margin-top: 10px;
  padding: 10px;
  font-size: 1rem;
  background-color: springgreen;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* Optional: Kein Umbruch auf kleinen Bildschirmen */
@media (max-width: 768px) {
  .ui-container {
    flex-wrap: nowrap;
  }
}
</style>