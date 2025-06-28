<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">👨‍👩‍👧‍👦 המתכונים המשפחתיים שלי</h2>

    <div v-if="familyRecipes.length">
      <div v-for="recipe in familyRecipes" :key="recipe.id" class="card mb-4">
        <div class="row g-0">
          <div class="col-md-4">
            <img
              :src="recipe.image"
              class="img-fluid rounded-start"
              alt="תמונה של מתכון"
            />
          </div>
          <div class="col-md-8">
            <div class="card-body">
              <h5 class="card-title">{{ recipe.title }}</h5>
              <p class="card-text">
                <strong>זמן הכנה:</strong> {{ recipe.readyInMinutes }} דקות
              </p>
              <p class="card-text">
                <strong>רכיבים:</strong>
                <ul>
                  <li v-for="(ingredient, index) in parseIngredients(recipe.extendedIngredients)" :key="index">
                    {{ ingredient }}
                  </li>
                </ul>
              </p>
              <p class="card-text">
                <strong>הוראות הכנה:</strong>
                <ol>
                  <li v-for="(step, idx) in parseInstructions(recipe.instructions)" :key="idx">
                    {{ step }}
                  </li>
                </ol>
              </p>
              <p class="card-text">
                <strong>הוכנס על ידי:</strong> {{ recipe.recipeMaster }}
              </p>
              <p class="card-text">
                <strong>מתי בשנה אוכלים אותו?</strong> {{ recipe.timeInYear }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="text-center text-muted mt-4">
      לא נמצאו מתכונים משפחתיים. כדאי להוסיף לפחות שלושה מתכונים הקשורים לבני משפחתך 💖
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, onMounted } from "vue";

export default {
  name: "FamilyRecipesPage",
  setup() {
    const familyRecipes = ref([]);

    const loadFamilyRecipes = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/family", {
          withCredentials: true,
        });
        familyRecipes.value = res.data.map((r) => ({
          ...r,
          id: r.recipeID,
        }));
      } catch (err) {
        console.error("שגיאה בטעינת מתכונים משפחתיים:", err);
      }
    };

    const parseIngredients = (raw) => {
      try {
        const parsed = JSON.parse(raw);
        return Array.isArray(parsed) ? parsed : [raw];
      } catch (e) {
        return raw.split(",").map((s) => s.trim());
      }
    };

    const parseInstructions = (text) => {
    if (!text) return [];
    return text
      .split(/\n|\r|\d+\.\s?/) // פיצול לפי מספרים או שורות
      .map(line => line.trim())
      .filter(line => line.length > 0);
  };

    onMounted(() => {
      loadFamilyRecipes();
    });

    return {
      familyRecipes,
      parseIngredients,
      parseInstructions
    };
  },
};
</script>

<style scoped>
.card-title {
  font-size: 1.5rem;
  font-weight: bold;
}
.card-text {
  margin-bottom: 0.5rem;
}
</style>
