<script setup lang="ts">
import { ref, computed } from 'vue'
import ingredientsData from '../data/ingredients.json'

interface Ingredient {
  id: number
  name: string
  calories: number
  macros: {
    proteines: number
    glucides: number
    lipides: number
  }
  portion: number
  unite: string
}

interface ShakerIngredient extends Ingredient {
  quantity: number
}

const initialIngredients = [
  { id: 1, name: "Lait demi-écrémé", quantity: 300 },
  { id: 2, name: "Beurre de cacahuète", quantity: 50 },
  { id: 3, name: "Flocons d'avoine", quantity: 100 },
  { id: 4, name: "Banane", quantity: 70 },
  { id: 5, name: "Miel", quantity: 10 }
]

const shakerIngredients = ref<ShakerIngredient[]>(
  initialIngredients.map(init => {
    const ingredient = ingredientsData.ingredients.find(i => i.id === init.id)!
    return {
      ...ingredient,
      quantity: init.quantity
    }
  })
)

const quantities = ref<Record<number, number>>({})

const totalCalories = computed(() => {
  return shakerIngredients.value.reduce((total, item) => {
    return total + (item.calories * item.quantity) / item.portion
  }, 0)
})

const totalMacros = computed(() => {
  return shakerIngredients.value.reduce(
    (total, item) => {
      return {
        proteines: total.proteines + (item.macros.proteines * item.quantity) / item.portion,
        glucides: total.glucides + (item.macros.glucides * item.quantity) / item.portion,
        lipides: total.lipides + (item.macros.lipides * item.quantity) / item.portion
      }
    },
    { proteines: 0, glucides: 0, lipides: 0 }
  )
})

const addIngredient = (ingredient: Ingredient) => {
  const quantity = quantities.value[ingredient.id] || 0
  if (quantity > 0) {
    const existing = shakerIngredients.value.find(item => item.id === ingredient.id)
    if (existing) {
      existing.quantity += quantity
    } else {
      shakerIngredients.value.push({
        ...ingredient,
        quantity
      })
    }
    quantities.value[ingredient.id] = 0
  }
}

const removeIngredient = (index: number) => {
  shakerIngredients.value.splice(index, 1)
}
</script>

<template>
  <div class="shaker-calculator">
    <div class="ingredients-summary">
      <h3>Ingrédients ajoutés</h3>
      <div class="ingredients-list">
        <div v-for="(ingredient, index) in shakerIngredients" 
             :key="index" 
             class="ingredient-item">
          <div class="ingredient-info">
            <span class="ingredient-name">{{ ingredient.name }}</span>
            <span class="ingredient-quantity">{{ ingredient.quantity }}{{ ingredient.unite }}</span>
          </div>
          <button class="delete-btn" @click="() => removeIngredient(index)">
            Supprimer
          </button>
        </div>
      </div>
      
      <div class="totals">
        <h4>Total:</h4>
        <div class="macro-grid">
          <div class="macro-item">
            <span class="macro-label">Calories:</span>
            <span class="macro-value">{{ Math.round(totalCalories) }}kcal</span>
          </div>
          <div class="macro-item">
            <span class="macro-label">Protéines:</span>
            <span class="macro-value">{{ Math.round(totalMacros.proteines) }}g</span>
          </div>
          <div class="macro-item">
            <span class="macro-label">Glucides:</span>
            <span class="macro-value">{{ Math.round(totalMacros.glucides) }}g</span>
          </div>
          <div class="macro-item">
            <span class="macro-label">Lipides:</span>
            <span class="macro-value">{{ Math.round(totalMacros.lipides) }}g</span>
          </div>
        </div>
      </div>
    </div>

    <div class="ingredients-cards">
      <h3>Ingrédients disponibles</h3>
      <div class="cards-grid">
        <div v-for="ingredient in ingredientsData.ingredients" 
             :key="ingredient.id" 
             class="ingredient-card">
          <h4>{{ ingredient.name }}</h4>
          <div class="macro-info">
            <p class="calories">{{ ingredient.calories }}kcal/{{ ingredient.portion }}{{ ingredient.unite }}</p>
            <div class="macros">
              <p>Protéines</p>
              <p>🥩 {{ ingredient.macros.proteines }}g</p>
              <p>Glucides</p>
              <p>🍚 {{ ingredient.macros.glucides }}g</p>
              <p>Lipides</p>
              <p>🥑 {{ ingredient.macros.lipides }}g</p>
            </div>
          </div>
          <div class="card-actions">
            <input type="number" 
                   v-model="quantities[ingredient.id]" 
                   :placeholder="ingredient.unite" />
            <button class="add-btn" @click="() => addIngredient(ingredient)">Ajouter</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.shaker-calculator {
  display: flex;
  gap: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  color: #e1e1e1;
}

.ingredients-summary, .ingredients-cards {
  flex: 1;
  background: #2a2a2a;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

h3 {
  color: #42b883;
  margin-bottom: 1.5rem;
  font-size: 1.5rem;
  text-align: center;
}

.ingredient-item {
  background: #333;
  margin-bottom: 10px;
  padding: 12px;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: transform 0.2s ease;
}

.ingredient-item:hover {
  transform: translateX(5px);
}

.ingredient-info {
  display: flex;
  justify-content: space-between;
  flex: 1;
  margin-right: 10px;
}

.ingredient-name {
  font-weight: 500;
}

.ingredient-quantity {
  color: #42b883;
}

.totals {
  margin-top: 2rem;
  padding: 1.5rem;
  background: #333;
  border-radius: 8px;
}

.macro-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 1rem;
  margin-top: 1rem;
}

.macro-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.macro-label {
  color: #888;
  font-size: 0.9rem;
}

.macro-value {
  color: #42b883;
  font-weight: bold;
  font-size: 1.1rem;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.ingredient-card {
  background: #333;
  padding: 1rem;
  border-radius: 8px;
  transition: transform 0.2s ease;
}

.ingredient-card:hover {
  transform: translateY(-5px);
}

.ingredient-card h4 {
  color: #42b883;
  margin-bottom: 0.5rem;
}

.macro-info {
  margin: 1rem 0;
}

.calories {
  color: #ff9800;
  font-weight: bold;
}

.macros {
  display: flex;
  justify-content: space-between;
  margin-top: 0.5rem;
}

.macros p {
  font-size: 0.9rem;
  color: #bbb;
}

.card-actions {
  display: flex;
  gap: 0.5rem;
  margin-top: 1rem;
}

input {
  width: 80px;
  padding: 8px;
  border: none;
  border-radius: 4px;
  background: #444;
  color: white;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.add-btn {
  background: #42b883;
  color: white;
  flex: 1;
}

.delete-btn {
  background: #ff4757;
  color: white;
  font-size: 0.9rem;
}

button:hover {
  opacity: 0.9;
  transform: scale(1.05);
}

/* Animations */
.ingredient-item, .ingredient-card {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style> 