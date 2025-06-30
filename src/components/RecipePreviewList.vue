<template>
  <div class="recipe-list-container">
    <div class="recipe-grid">
      <RecipePreview
        v-for="r in recipes"
        :key="r.id"
        :recipe="r"
        @update-favorite="(id, isFav) => $emit('update-favorite', id, isFav)"
        @refresh="$emit('refresh')"
      />
    </div>
  </div>
</template>

<script>
import RecipePreview from "./RecipePreview.vue";
export default {
  name: "RecipePreviewList",
  components: { RecipePreview },
  props: {
    recipes: { type: Array, required: true }
  }
};
</script>

<style scoped>
.recipe-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1rem;
  justify-content: center;
}

</style>
