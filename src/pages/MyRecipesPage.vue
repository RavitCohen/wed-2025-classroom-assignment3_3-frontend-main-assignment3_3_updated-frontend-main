<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">🍳 המתכונים שלי</h2>

    <div v-if="myRecipes.length">
      <RecipePreviewList :recipes="myRecipes" title="מתכונים שיצרתי" />
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

    onMounted(() => {
      loadMyRecipes();
    });

    return { myRecipes };
  }
};
</script>
