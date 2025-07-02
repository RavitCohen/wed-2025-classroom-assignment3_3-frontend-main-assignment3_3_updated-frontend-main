<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">🍳 המתכונים שלי</h2>

    <div v-if="myRecipes.length">
      <RecipePreviewList
        :recipes="myRecipes"
        title="מתכונים שיצרתי"
        @update-favorite="handleFavoriteUpdate"
        @delete-recipe="handleDeleteRecipe"
      />
    </div>
    <div v-else class="text-center text-muted mt-4">
      עדיין לא יצרת אף מתכון. נסה להוסיף אחד דרך כפתור "מתכון חדש" 👩‍🍳
    </div>
  </div>
</template>

<script>
import RecipePreviewList from "../components/RecipePreviewList.vue";
import axios from "axios";
import { ref, onMounted } from "vue";

export default {
  name: "MyRecipesPage",
  components: { RecipePreviewList },
  setup() {
    const myRecipes = ref([]);

    const loadMyRecipes = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/recipes", {
          withCredentials: true,
        });
        // מסננים רק מתכונים עם ID שמתחיל ב-U_
        myRecipes.value = res.data.filter(r => r.recipeID && r.recipeID.startsWith("U_"));
      } catch (err) {
        console.error("שגיאה בטעינת מתכונים שלי:", err);
      }
    };

    const handleFavoriteUpdate = (recipeId, isNowFavorite) => {
      const recipe = myRecipes.value.find(r => r.recipeID === recipeId);
      if (recipe) {
        recipe.isFavoriteByUser = isNowFavorite;
      }
    };

    const handleDeleteRecipe = async (recipeId) => {
      if (confirm("האם למחוק את המתכון הזה?")) {
        try {
          await axios.delete(`http://localhost:3000/user/recipes/${recipeId.replace("U_", "")}`, {
            withCredentials: true,
          });
          myRecipes.value = myRecipes.value.filter(r => r.recipeID !== recipeId);
        } catch (err) {
          console.error("שגיאה במחיקת מתכון:", err);
        }
      }
    };

    onMounted(() => {
      loadMyRecipes();
    });

    return {
      myRecipes,
      handleFavoriteUpdate,
      handleDeleteRecipe
    };
  }
};
</script>
