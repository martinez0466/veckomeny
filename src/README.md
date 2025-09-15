# Weekly Meal Planner (Vue + Vite)

Labb 2 – HTML, CSS och Vue. A simple weekly meal planner built with **Vue 3 (Composition API)** and **Vite**.

## Features
- Add/remove meals stored in an object (`planner.meals`)
- User profile (name, diet) with `v-if` gated planning area
- Validation (empty input, per-day duplicate check)
- Sorted view Monday→Sunday + total counter
- LocalStorage persistence (`user` + `meals`)
- Reset profile & clear all meals (with confirmations)
- Green, food-friendly theme and hero image (`public/food.jpg` or Unsplash fallback)

## Run locally
```bash
npm install
npm run dev
