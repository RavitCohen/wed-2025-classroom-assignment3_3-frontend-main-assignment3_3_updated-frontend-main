<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">❤️ המועדפים שלי</h2>
    <RecipePreviewList
      v-if="favorites.length > 0"
      :recipes="favorites"
      title="מתכונים שמורים"
      @update-favorite="handleFavoriteUpdate"
    />
    <div v-else class="text-center text-muted mt-3">
      אין מתכונים שמורים כרגע.
    </div>
  </div>
</template>

<script>
import RecipePreviewList from "../components/RecipePreviewList.vue";
import axios from "axios";
import { ref, onMounted } from "vue";

export default {
  name: "FavoritesPage",
  components: { RecipePreviewList },
  setup() {
    const favorites = ref([]);

    const loadFavorites = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/favorites", {
          withCredentials: true,
        });
        favorites.value = res.data;
      } catch (err) {
        console.error("שגיאה בטעינת מועדפים:", err);
      }
    };

    const handleFavoriteUpdate = (recipeId, isNowFavorite) => {
      if (!isNowFavorite) {
        favorites.value = favorites.value.filter(r => r.id !== recipeId);
      }
    };

    onMounted(() => {
      loadFavorites();
    });

    return { favorites, handleFavoriteUpdate };
  }
};
</script>