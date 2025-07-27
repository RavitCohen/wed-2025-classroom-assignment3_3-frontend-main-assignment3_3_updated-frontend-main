<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">👨‍👩‍👧‍👦 המתכונים המשפחתיים שלי</h2>

    <div v-if="familyRecipes.length">
    <div v-for="recipe in familyRecipes" :key="recipe.id" class="recipe-card mb-5">
          <div class="recipe-content">
              <div class="image-carousel">
              <img
                v-for="(imgSrc, index) in recipe.image"
                :key="index"
                :src="`/${imgSrc}`"
                class="recipe-image"
                alt="תמונה של מתכון"
              />
            </div>
              <div class="recipe-details">
                <h5 class="card-title">{{ recipe.title }}</h5>
                <p style="font-size: 1.2rem;"><strong>⏱ זמן הכנה:</strong> {{ recipe.readyInMinutes }} דקות</p>
                <p style="font-size: 1.5rem;"><strong>🧾 רכיבים:</strong></p>
                <ul>
                  <li v-for="(ingredient, index) in parseIngredients(recipe.extendedIngredients)" :key="index">
                    {{ ingredient }}
                  </li>
                </ul>
                <p class="instructions" style="font-size: 1.5rem;"><strong>📋 הוראות הכנה:</strong></p>
                <ol>
                  <li v-for="(step, idx) in parseInstructions(recipe.instructions)" :key="idx">
                    {{ step }}
                  </li>
                </ol>
                <p class="author"><strong>👩‍🍳 השף במשפחה שמתמחה במתכון:</strong> {{ recipe.recipeMaster }}</p>
                <p class="season"><strong>🎉 מתי בשנה אוכלים אותו?</strong> {{ recipe.timeInYear }}</p>
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
        console.log(`img: ${res.data[0].image}`);
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
      .split(/\n|\r|\d+\.\s?/) 
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
.recipe-card {
  background-color: #ffffff;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  max-width: 2500px; 
  margin: auto;
  transition: 0.3s ease;
}

.recipe-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.image-carousel {
  display: flex;
  flex-direction: row;
  overflow-x: auto ;
  gap: 1rem;
  white-space: nowrap;  
  padding-bottom: 1rem;
  scroll-behavior: smooth;
}

.recipe-image {
  flex: 0 0 auto;
  width: 400px;          
  height: 300px;        
  object-fit: cover;     
  border-radius: 12px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
  transition: transform 0.3s ease-in-out;
}


.recipe-image:hover {
  transform: scale(1.05);
  cursor: pointer;
}

.recipe-details {
  font-size: 1.5rem;
  line-height: 1.6;
  text-align: right;
}

.recipe-details .author {
  font-style: italic;
  font-weight: 500;
  color: #9a1b4c;
  margin-top: 1rem;
  font-size: 1.5rem;
}

.recipe-details .season {
  font-style: italic;
  font-weight: bold;
  color: #F57C00;
  font-size: 1.5rem;
}
.recipe-details ul,
.recipe-details ol {
  padding-inline-start: 1.5rem;
  margin-top: 0.5rem;
  margin-bottom: 1rem;
  line-height: 1.7;
  color: #333;
}

.recipe-details li {
  margin-bottom: 0.4rem;
}
.recipe-details h5 {
  font-size: 1.8rem;
  font-weight: bold;
  color: #263238;
  margin-bottom: 1rem;
}

.recipe-details p {
  font-size: 1.1rem;
  line-height: 1.8;
  margin-bottom: 0.8rem;
  color: #424242;
}
.recipe-details strong {
  color: #37474F; 
  font-weight: 700;
}

.recipe-details em {
  color: #607D8B;
  font-style: normal;
}



</style>
