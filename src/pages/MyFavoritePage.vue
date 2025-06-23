<template>
  <div class="container py-4">
    <h2 class="text-center mb-4">❤️ המועדפים שלי</h2>
    <RecipePreviewList v-if="favorites.length > 0" :recipes="favorites" title="מתכונים שמורים" />
    <div v-else class="text-center text-muted mt-3">
      אין מתכונים שמורים כרגע.
    </div>
  </div>
</template>

<script>
import RecipePreviewList from "../components/RecipePreviewList.vue";
import axios from "axios";
import store from "../store";
import { ref, onMounted } from "vue";

export default {
  name: "FavoritesPage",
  components: { RecipePreviewList },
  setup() {
    const favorites = ref([]);

    const loadFavorites = async () => {
      try {
        const res = await axios.get("http://localhost:3000/user/favorites", {
          headers: { Authorization: `Bearer ${store.token}` }
        });
        favorites.value = res.data.filter(recipe => recipe.isFavorite);
      } catch (err) {
        console.error("שגיאה בטעינת מועדפים:", err);
      }
    };

    onMounted(() => {
      loadFavorites();
    });

    return { favorites };
  }
};
</script>
