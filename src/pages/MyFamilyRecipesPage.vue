<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">👨‍👩‍👧‍👦 המתכונים המשפחתיים שלי</h2>

    <div v-if="familyRecipes.length">
      <RecipePreviewList :recipes="familyRecipes" title="מתכונים משפחתיים" />
    </div>
    <div v-else class="text-center text-muted mt-4">
      לא נמצאו מתכונים משפחתיים. כדאי להוסיף לפחות שלושה מתכונים הקשורים לבני משפחתך 💖
    </div>
  </div>
</template>

<script>
import RecipePreviewList from "../components/RecipePreviewList.vue";
import axios from "axios";
import store from "../store";
import { ref, onMounted } from "vue";

export default {
  name: "FamilyRecipesPage",
  components: { RecipePreviewList },
  setup() {
    const familyRecipes = ref([]);

    const loadFamilyRecipes = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/family-recipes", {
          headers: { Authorization: `Bearer ${store.token}` }
        });
        familyRecipes.value = res.data;
      } catch (err) {
        console.error("שגיאה בטעינת מתכונים משפחתיים:", err);
      }
    };

    onMounted(() => {
      loadFamilyRecipes();
    });

    return { familyRecipes };
  }
};
</script>
