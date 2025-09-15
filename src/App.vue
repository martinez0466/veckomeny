<script setup>
import { reactive, ref, computed } from 'vue'

// Användarinfo/inställningar (se krav)
const user = reactive({ name: '', diet: 'none' })

// Objekt som innehåller listan (krav: lista i objekt)
const planner = reactive({
  meals: [{ id: crypto.randomUUID(), day: 'Måndag', dish: 'Pasta pesto' }],
})

const newDay = ref('Måndag')
const newDish = ref('')

const greeting = computed(() => (user.name ? `Hej, ${user.name}!` : 'Välkommen!'))
const canPlan = computed(() => user.name.trim().length > 0)

function addMeal() {
  const dish = newDish.value.trim()
  if (!dish) return
  planner.meals.push({ id: crypto.randomUUID(), day: newDay.value, dish })
  newDish.value = ''
}

function removeMeal(id) {
  planner.meals = planner.meals.filter(m => m.id !== id)
}
</script>

<template>
  <main class="container">
    <section class="card">
      <h1 class="section-title">Veckomeny</h1>
      <p class="help">Planera dina måltider för veckan. Lägg till rätter, redigera och ta bort.</p>
    </section>

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

    <section class="card" v-if="canPlan">
      <h2 class="section-title">Lägg till rätt</h2>
      <form @submit.prevent="addMeal">
        <div class="row row-2">
          <div>
            <label class="label" for="day">Dag</label>
            <select id="day" v-model="newDay">
              <option>Måndag</option><option>Tisdag</option><option>Onsdag</option>
              <option>Torsdag</option><option>Fredag</option><option>Lördag</option><option>Söndag</option>
            </select>
          </div>
          <div>
            <label class="label" for="dish">Rätt</label>
            <input id="dish" v-model="newDish" placeholder="t.ex. Tacos" />
          </div>
        </div>
        <div style="margin-top:12px;">
          <button type="submit">Lägg till</button>
        </div>
      </form>
    </section>

    <section class="card" v-if="canPlan">
      <h2 class="section-title">Veckans rätter</h2>
      <p v-if="planner.meals.length === 0" class="help">Inga rätter ännu.</p>
      <ul style="list-style:none; padding:0; margin:0;">
        <li v-for="m in planner.meals" :key="m.id"
            style="display:flex; align-items:center; justify-content:space-between; border:1px solid rgba(255,255,255,.06); border-radius:12px; padding:10px; margin-bottom:8px; background:#0c1322;">
          <div><strong>{{ m.day }}:</strong> {{ m.dish }}</div>
          <button class="danger" @click="removeMeal(m.id)">Ta bort</button>
        </li>
      </ul>
    </section>

    <section class="card" v-else>
      <h2 class="section-title">Kom igång</h2>
      <p class="help">Fyll i ditt namn ovan för att börja planera din veckomeny.</p>
    </section>
  </main>
</template>

<style scoped></style>
