<script setup>
import { reactive, ref, computed, watch } from 'vue'

/**
 * Användaruppgifter / inställningar (se kurskrav)
 */
const user = reactive({
  name: '',
  diet: 'none', // none | vegetarian | vegan | pescetarian
})

/**
 * Objekt som innehåller listan (KRAV: listan sparas i ett objekt)
 */
const planner = reactive({
  meals: [
    // Exempelpost (kan tas bort)
    { id: crypto.randomUUID(), day: 'Måndag', dish: 'Pasta pesto' },
  ],
})

/**
 * Form-state
 */
const newDay = ref('Måndag')
const newDish = ref('')

/**
 * Validering / Hjälp-UI
 */
const formError = ref('')
const greeting = computed(() => (user.name ? `Hej, ${user.name}!` : 'Välkommen!'))
const canPlan = computed(() => user.name.trim().length > 0)              // styr v-if
const canSubmit = computed(() => newDish.value.trim().length > 0)        // disable-knapp

// Rensa fel när användaren ändrar fält
watch([newDish, newDay], () => { formError.value = '' })

/**
 * Metoder (KRAV: relevanta metoder/funktioner)
 */
function addMeal() {
  const dish = newDish.value.trim()

  // 1) Tomt fält
  if (!dish) {
    formError.value = 'Skriv in en rätt innan du lägger till.'
    return
  }

  // 2) Lätt dubblettkontroll för samma dag (case-insensitive)
  const duplicate = planner.meals.some(
    (m) => m.day === newDay.value && m.dish.toLowerCase() === dish.toLowerCase()
  )
  if (duplicate) {
    formError.value = 'Den rätten finns redan för vald dag.'
    return
  }

  planner.meals.push({ id: crypto.randomUUID(), day: newDay.value, dish })
  newDish.value = ''
  formError.value = ''
}

function removeMeal(id) {
  planner.meals = planner.meals.filter((m) => m.id !== id)
}
</script>

<template>
  <main class="container">
    <!-- Header -->
    <section class="card">
      <h1 class="section-title">Veckomeny</h1>
      <p class="help">Planera dina måltider för veckan. Lägg till rätter, redigera och ta bort.</p>
    </section>

    <!-- Användaruppgifter / Inställningar -->
    <section class="card">
      <h2 class="section-title">Användare & inställningar</h2>
      <div class="row row-2">
        <div>
          <label class="label" for="name">Ditt namn</label>
          <input id="name" v-model="user.name" placeholder="Skriv ditt namn" />
        </div>
        <div>
          <label class="label" for="diet">Kostpreferens</label>
          <select id="diet" v-model="user.diet">
            <option value="none">Ingen</option>
            <option value="vegetarian">Vegetarisk</option>
            <option value="vegan">Vegansk</option>
            <option value="pescetarian">Pescetarian</option>
          </select>
        </div>
      </div>
      <p class="help" style="margin-top:8px;">{{ greeting }}</p>
    </section>

    <!-- Planering (visas först när namn är ifyllt – KRAV: v-if) -->
    <section class="card" v-if="canPlan">
      <h2 class="section-title">Lägg till rätt</h2>
      <form @submit.prevent="addMeal">
        <div class="row row-2">
          <div>
            <label class="label" for="day">Dag</label>
            <select id="day" v-model="newDay">
              <option>Måndag</option>
              <option>Tisdag</option>
              <option>Onsdag</option>
              <option>Torsdag</option>
              <option>Fredag</option>
              <option>Lördag</option>
              <option>Söndag</option>
            </select>
          </div>
          <div>
            <label class="label" for="dish">Rätt</label>
            <input
              id="dish"
              v-model="newDish"
              placeholder="t.ex. Tacos"
              :aria-invalid="!!formError"
              aria-describedby="dish-help"
            />
          </div>
        </div>
        <div style="margin-top:12px; display:flex; gap:12px; align-items:center;">
          <button type="submit" :disabled="!canSubmit">Lägg till</button>
          <span id="dish-help" class="help" v-if="formError">{{ formError }}</span>
        </div>
      </form>
    </section>

    <!-- Lista (från objektet planner.meals – KRAV: lista i objekt) -->
    <section class="card" v-if="canPlan">
      <h2 class="section-title">Veckans rätter</h2>
      <p v-if="planner.meals.length === 0" class="help">Inga rätter ännu.</p>

      <ul style="list-style:none; padding:0; margin:0;">
        <li
          v-for="m in planner.meals"
          :key="m.id"
          style="display:flex; align-items:center; justify-content:space-between; border:1px solid rgba(255,255,255,.06); border-radius:12px; padding:10px; margin-bottom:8px; background:#0c1322;"
        >
          <div>
            <strong>{{ m.day }}:</strong> {{ m.dish }}
          </div>
          <button class="danger" @click="removeMeal(m.id)">Ta bort</button>
        </li>
      </ul>
    </section>

    <!-- Hjälpsektion när namn saknas -->
    <section class="card" v-else>
      <h2 class="section-title">Kom igång</h2>
      <p class="help">Fyll i ditt namn ovan för att börja planera din veckomeny.</p>
    </section>
  </main>
</template>

<style scoped>
/* Håll komponent-styling minimal – global stil finns i src/style.css */
</style>
