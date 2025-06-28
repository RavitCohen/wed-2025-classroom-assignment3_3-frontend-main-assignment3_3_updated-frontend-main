<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">🍳 המתכונים שלי</h2>

    <div v-if="myRecipes.length">
      <RecipePreviewList
        :recipes="myRecipes"
        title="מתכונים שיצרתי"
        @update-favorite="handleFavoriteUpdate"
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
        myRecipes.value = res.data;
      } catch (err) {
        console.error("שגיאה בטעינת מתכונים שלי:", err);
      }
    };

  //   const handleLikesUpdate = (recipeId) => {
  //   const recipe = myRecipes.value.find(r => r.id === recipeId);
  //   if (recipe) {
  //     recipe.popularity = (recipe.popularity || 0) + 1;
  //   }
  // };

    const handleFavoriteUpdate = (recipeId, isNowFavorite) => {
      const recipe = myRecipes.value.find(r => r.id === recipeId);
      if (recipe) {
        recipe.isFavoriteByUser = isNowFavorite;
      }
    };

    onMounted(() => {
      loadMyRecipes();
    });

    return {
      myRecipes,
      handleFavoriteUpdate
    };
  }
};
</script>
