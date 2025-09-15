<script setup>
/* =========  Imports  ========= */
import { reactive, ref, computed, watch, onMounted } from 'vue'

/* =========  Small utilities  ========= */
const uuid = () =>
  typeof crypto !== 'undefined' && crypto.randomUUID
    ? crypto.randomUUID()
    : Math.random().toString(36).slice(2)

/* Hero image:
   - Preferred: place your own image at /public/food.jpg
   - Fallback: a pleasant Unsplash image (auto-applied if /food.jpg missing)
*/
const HERO_LOCAL = '/food.jpg'
const HERO_FALLBACK =
  'https://images.unsplash.com/photo-1504674900247-0877df9cc836?q=80&w=1600&auto=format&fit=crop'  // CC-friendly preview

/* =========  State  ========= */
/** User profile / settings */
const user = reactive({
  name: '',
  diet: 'none', // none | vegetarian | vegan | pescetarian
})

/** Domain state: list stored inside an object (course requirement) */
const planner = reactive({
  meals: [
    // Example item (safe to delete)
    { id: uuid(), day: 'Monday', dish: 'Pesto pasta' },
  ],
})

/** Form state */
const newDay = ref('Monday')
const newDish = ref('')

/* =========  Computed / UI helpers  ========= */
const formError = ref('')
const greeting = computed(() => (user.name ? `Hi, ${user.name}!` : 'Welcome!'))
const canPlan = computed(() => user.name.trim().length > 0)
const canSubmit = computed(() => newDish.value.trim().length > 0)

/** Sorted view of meals by weekday order */
const WEEKDAYS_EN = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
const sortedMeals = computed(() =>
  [...planner.meals].sort((a, b) => WEEKDAYS_EN.indexOf(a.day) - WEEKDAYS_EN.indexOf(b.day))
)

/* =========  Persistence (localStorage)  ========= */
const STORAGE_KEYS = { user: 'mp_user_v1', meals: 'mp_meals_v1' }

function loadState() {
  try {
    const u = localStorage.getItem(STORAGE_KEYS.user)
    if (u) {
      const parsed = JSON.parse(u)
      if (parsed && typeof parsed === 'object') {
        user.name = parsed.name ?? ''
        user.diet = parsed.diet ?? 'none'
      }
    }
  } catch (_) {}

  try {
    const m = localStorage.getItem(STORAGE_KEYS.meals)
    if (m) {
      const parsed = JSON.parse(m)
      if (Array.isArray(parsed)) {
        planner.meals = parsed
          .filter((it) => it && it.id && it.day && it.dish)
          .map((it) => ({ id: it.id, day: it.day, dish: it.dish }))
      }
    }
  } catch (_) {}
}

onMounted(loadState)

watch(
  user,
  (val) => {
    try { localStorage.setItem(STORAGE_KEYS.user, JSON.stringify(val)) } catch (_) {}
  },
  { deep: true }
)

watch(
  () => planner.meals,
  (val) => {
    try { localStorage.setItem(STORAGE_KEYS.meals, JSON.stringify(val)) } catch (_) {}
  },
  { deep: true }
)

/* =========  Methods  ========= */
function addMeal() {
  const dish = newDish.value.trim()
  if (!dish) { formError.value = 'Please enter a dish before adding.'; return }
  const duplicate = planner.meals.some(
    (m) => m.day === newDay.value && m.dish.toLowerCase() === dish.toLowerCase()
  )
  if (duplicate) { formError.value = 'That dish already exists for the selected day.'; return }
  planner.meals.push({ id: uuid(), day: newDay.value, dish })
  newDish.value = ''
  formError.value = ''
}

function removeMeal(id) {
  planner.meals = planner.meals.filter((m) => m.id !== id)
}

/** Clear all meals (with confirmation) */
function clearMeals() {
  if (planner.meals.length === 0) return
  const ok = confirm('Remove ALL meals? This cannot be undone.')
  if (!ok) return
  planner.meals = []
}

/** Reset profile (user + meals + storage) */
function resetProfile() {
  const ok = confirm('Reset your name, diet, and all meals? This cannot be undone.')
  if (!ok) return
  user.name = ''
  user.diet = 'none'
  planner.meals = []
  try {
    localStorage.removeItem(STORAGE_KEYS.user)
    localStorage.removeItem(STORAGE_KEYS.meals)
  } catch (_) {}
}
</script>

<template>
  <main class="container">
    <!-- Header -->
    <section class="card">
      <h1 class="section-title">Weekly Meal Planner</h1>
      <p class="help">Plan your meals for the week. Add, edit, and remove dishes.</p>
      <p class="help" style="margin-top:8px;">{{ greeting }}</p>

      <!-- Nice hero image -->
      <img
        class="hero-img"
        :src="HERO_LOCAL"
        :alt="`Delicious food hero image`"
        @error="(e) => (e.target.src = HERO_FALLBACK)"
        style="margin-top:12px;"
      />
    </section>

    <!-- User & settings -->
    <section class="card">
      <h2 class="section-title">User & Settings</h2>
      <div class="row row-2">
        <div>
          <label class="label" for="name">Your name</label>
          <input id="name" v-model="user.name" placeholder="Type your name" autofocus />
        </div>
        <div>
          <label class="label" for="diet">Diet preference</label>
          <select id="diet" v-model="user.diet">
            <option value="none">None</option>
            <option value="vegetarian">Vegetarian</option>
            <option value="vegan">Vegan</option>
            <option value="pescetarian">Pescetarian</option>
          </select>
        </div>
      </div>

      <!-- Profile utilities -->
      <div style="margin-top:12px; display:flex; gap:12px; flex-wrap:wrap;">
        <button class="danger" @click="resetProfile">Reset profile</button>
      </div>
    </section>

    <!-- Planning (shown only when a name is provided) -->
    <section class="card" v-if="canPlan">
      <h2 class="section-title">Add meal</h2>
      <form @submit.prevent="addMeal">
        <div class="row row-2">
          <div>
            <label class="label" for="day">Day</label>
            <select id="day" v-model="newDay">
              <option>Monday</option>
              <option>Tuesday</option>
              <option>Wednesday</option>
              <option>Thursday</option>
              <option>Friday</option>
              <option>Saturday</option>
              <option>Sunday</option>
            </select>
          </div>
          <div>
            <label class="label" for="dish">Dish</label>
            <input
              id="dish"
              v-model="newDish"
              placeholder="e.g., Tacos"
              :aria-invalid="!!formError"
              aria-describedby="dish-help"
            />
          </div>
        </div>
        <div style="margin-top:12px; display:flex; gap:12px; align-items:center; flex-wrap:wrap;">
          <button type="submit" :disabled="!canSubmit">Add</button>
          <span id="dish-help" class="help" v-if="formError">{{ formError }}</span>
        </div>
      </form>
    </section>

    <!-- Meals list (from object `planner.meals`) -->
    <section class="card" v-if="canPlan">
      <h2 class="section-title">
        Meals this week <small class="help">({{ planner.meals.length }})</small>
      </h2>

      <p v-if="planner.meals.length === 0" class="help">No meals yet.</p>

      <ul style="list-style:none; padding:0; margin:0;">
        <li
          v-for="m in sortedMeals"
          :key="m.id"
          style="display:flex; align-items:center; justify-content:space-between; border:1px solid rgba(255,255,255,.06); border-radius:12px; padding:10px; margin-bottom:8px; background:#0e1a13;"
        >
          <div>
            <strong>{{ m.day }}:</strong> {{ m.dish }}
          </div>
          <div style="display:flex; gap:8px;">
            <button class="danger" @click="removeMeal(m.id)">Remove</button>
          </div>
        </li>
      </ul>

      <!-- List utility actions -->
      <div style="margin-top:12px;">
        <button class="danger" @click="clearMeals" :disabled="planner.meals.length === 0">
          Clear all meals
        </button>
      </div>
    </section>

    <!-- Onboarding when name is missing -->
    <section class="card" v-else>
      <h2 class="section-title">Get started</h2>
      <p class="help">Enter your name above to start planning your weekly meals.</p>
    </section>
  </main>
</template>

<style scoped>
/* Keep component-level styling minimal – global rules are in src/style.css */
</style>
