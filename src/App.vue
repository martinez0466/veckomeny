<script setup>
/* =========  Imports  ========= */
import { reactive, ref, computed, watch, onMounted } from 'vue'

/* =========  Small utilities  ========= */
const uuid = () =>
  typeof crypto !== 'undefined' && crypto.randomUUID
    ? crypto.randomUUID()
    : Math.random().toString(36).slice(2)

/* Hero image sources */
const HERO_LOCAL = '/food.jpg'
const HERO_FALLBACK =
  'https://images.unsplash.com/photo-1504674900247-0877df9cc836?q=80&w=1600&auto=format&fit=crop'

/* =========  State  ========= */
/** User profile / settings */
const user = reactive({
  name: '',
  diet: 'none', // none | vegetarian | vegan | pescetarian
})

/** Domain state: list stored inside an object (course requirement) */
const planner = reactive({
  meals: [
    { id: uuid(), day: 'Monday', dish: 'Pesto pasta', priority: 'normal' }, // example
  ],
})

/** Form state */
const newDay = ref('Monday')
const newDish = ref('')
const newPriority = ref('normal') // low | normal | high

/** View state (custom functionality) */
const compactMode = ref(false)     // hides hero, tightens list
const prioritizeHigh = ref(true)   // bring high-priority items to the top

/** Search (optional helper) */
const query = ref('')

/* =========  Computed / UI helpers  ========= */
const formError = ref('')
const greeting = computed(() => (user.name ? `Hi, ${user.name}!` : 'Welcome!'))
const canPlan = computed(() => user.name.trim().length > 0)
const canSubmit = computed(() => newDish.value.trim().length > 0)

/** Filtering + sorting */
const WEEKDAYS_EN = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday']
const PRIO_ORDER = { high: 0, normal: 1, low: 2 }

const filteredMeals = computed(() => {
  const q = query.value.trim().toLowerCase()
  if (!q) return planner.meals
  return planner.meals.filter(
    (m) => m.dish.toLowerCase().includes(q) || m.day.toLowerCase().includes(q)
  )
})

const visibleMeals = computed(() => {
  const list = [...filteredMeals.value]
  return list.sort((a, b) => {
    // priority-first if toggle is ON
    if (prioritizeHigh.value) {
      const pr = (PRIO_ORDER[a.priority] ?? 1) - (PRIO_ORDER[b.priority] ?? 1)
      if (pr !== 0) return pr
    }
    // then weekday order
    return WEEKDAYS_EN.indexOf(a.day) - WEEKDAYS_EN.indexOf(b.day)
  })
})

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
          .map((it) => ({
            id: it.id,
            day: it.day,
            dish: it.dish,
            priority: it.priority ?? 'normal',
          }))
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
    (m) =>
      m.day === newDay.value &&
      m.dish.toLowerCase() === dish.toLowerCase() &&
      m.priority === newPriority.value
  )
  if (duplicate) { formError.value = 'That dish with the same priority already exists for that day.'; return }

  planner.meals.push({ id: uuid(), day: newDay.value, dish, priority: newPriority.value })
  newDish.value = ''
  newPriority.value = 'normal'
  formError.value = ''
}

function removeMeal(id) {
  planner.meals = planner.meals.filter((m) => m.id !== id)
}

function clearMeals() {
  if (planner.meals.length === 0) return
  const ok = confirm('Remove ALL meals? This cannot be undone.')
  if (!ok) return
  planner.meals = []
}

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

      <!-- Hero (hidden in compact mode, and auto-hidden on small screens via CSS) -->
      <img
        v-if="!compactMode"
        class="hero-img"
        :src="HERO_LOCAL"
        alt="Delicious food hero image"
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

      <!-- View toggles -->
      <div style="margin-top:12px; display:flex; gap:12px; flex-wrap:wrap;">
        <button class="secondary" @click="compactMode = !compactMode">
          {{ compactMode ? 'Exit compact mode' : 'Enter compact mode' }}
        </button>
        <button class="secondary" @click="prioritizeHigh = !prioritizeHigh">
          {{ prioritizeHigh ? 'Disable prioritize-high' : 'Enable prioritize-high' }}
        </button>
        <button class="danger" @click="resetProfile">Reset profile</button>
      </div>
    </section>

    <!-- Add meal -->
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

        <div class="row" style="margin-top:8px;">
          <div>
            <label class="label" for="prio">Priority</label>
            <select id="prio" v-model="newPriority">
              <option value="low">Low</option>
              <option value="normal">Normal</option>
              <option value="high">High</option>
            </select>
          </div>
        </div>

        <div style="margin-top:12px; display:flex; gap:12px; align-items:center; flex-wrap:wrap;">
          <button type="submit" :disabled="!canSubmit">Add</button>
          <span id="dish-help" class="help" v-if="formError">{{ formError }}</span>
        </div>
      </form>
    </section>

    <!-- Meals list -->
    <section class="card" v-if="canPlan">
      <h2 class="section-title">
        Meals this week
        <small class="help">
          ({{ query ? `${visibleMeals.length} / ${planner.meals.length}` : planner.meals.length }})
        </small>
      </h2>

      <!-- Search -->
      <div class="row" style="margin:8px 0 12px;">
        <div>
          <label class="label" for="search">Search meals</label>
          <input id="search" v-model="query" placeholder="Type dish or day (e.g., Friday, tacos)" />
        </div>
      </div>

      <!-- Empty states -->
      <p v-if="planner.meals.length === 0" class="help">No meals yet.</p>
      <p v-else-if="visibleMeals.length === 0" class="help">No results for “{{ query }}”.</p>

      <!-- List -->
      <ul v-else style="list-style:none; padding:0; margin:0;">
        <li
          v-for="m in visibleMeals"
          :key="m.id"
          :class="['meal-item', `prio-${m.priority}`, { compact: compactMode }]"
        >
          <div>
            <strong>{{ m.day }}:</strong> {{ m.dish }}
            <span class="pill" :class="{
              'pill-high': m.priority==='high',
              'pill-normal': m.priority==='normal',
              'pill-low': m.priority==='low'
            }">{{ m.priority }}</span>
          </div>
          <div style="display:flex; gap:8px;">
            <button class="danger" @click="removeMeal(m.id)">Remove</button>
          </div>
        </li>
      </ul>

      <!-- List actions -->
      <div style="margin-top:12px;">
        <button class="danger" @click="clearMeals" :disabled="planner.meals.length === 0">
          Clear all meals
        </button>
      </div>
    </section>

    <!-- Onboarding -->
    <section class="card" v-else>
      <h2 class="section-title">Get started</h2>
      <p class="help">Enter your name above to start planning your weekly meals.</p>
    </section>
  </main>
</template>

<style scoped>
</style>
