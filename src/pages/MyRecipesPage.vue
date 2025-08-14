<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">🍳 המתכונים שלי</h2>

    <div v-if="note" class="note">{{ note }}</div>

    <div v-if="myRecipes.length">
      <RecipePreviewList
        :recipes="myRecipes"
        :isShowDelete="true"
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
    const note = ref("");

    const loadMyRecipes = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/recipes", {
          withCredentials: true,
        });
        myRecipes.value = res.data;
      } catch (err) {
        console.error("שגיאה בטעינת מתכונים שלי:", err);
        showNote("שגיאה בטעינת המתכונים 😕");
      }
    };

    const showNote = (text, ms = 2500) => {
      note.value = text;
      clearTimeout(showNote.t);
      showNote.t = setTimeout(() => (note.value = ""), ms);
    };

    const handleFavoriteUpdate = (recipeId, isNowFavorite) => {
      const recipe = myRecipes.value.find(r => r.recipeID === recipeId);
      if (recipe) {
        recipe.isFavoriteByUser = isNowFavorite;
      }
    };

    const handleDeleteRecipe = async (recipeId) => {
      myRecipes.value = myRecipes.value.filter(r => r.recipeID !== recipeId);
      showNote("המתכון הוסר מהמתכונים שלי ✅");
      await loadMyRecipes();
    };

    onMounted(() => {
      loadMyRecipes();
    });

    return {
      myRecipes,
      note,
      handleFavoriteUpdate,
      handleDeleteRecipe
    };
  }
};
</script>

<style scoped>

.note {
  background: #f1fff6;
  border: 1px solid #b7e4c7;
  color: #14532d;
  border-radius: 10px;
  padding: .5rem .75rem;
  font-size: .95rem;
  margin-bottom: .75rem;
}
</style>
