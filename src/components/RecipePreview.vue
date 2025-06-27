<template>
  <router-link :to="{ name: 'recipe', params: { recipeId: recipe.id } }" class="card-link">
    <div class="card h-100 recipe-card">
      <img
        v-if="recipe.image"
        :src="recipe.image"
        class="card-img-top recipe-image"
        alt="Recipe image"
      />
      <div class="card-body text-center">
        <h5 class="card-title">{{ recipe.title }}</h5>
        <p class="card-text">{{ recipe.readyInMinutes }} דקות</p>
        <p class="card-text">{{ recipe.popularity }} לייקים</p>

        <div class="badges my-2">
          <b-badge v-if="recipe.vegetarian" variant="success" class="mx-1">🌱 טבעוני</b-badge>
          <b-badge v-if="recipe.glutenFree" variant="warning" class="mx-1">🚫 גלוטן</b-badge>
          <b-badge v-if="recipe.isWatched" variant="info" class="mx-1">👁️ נצפה</b-badge>
          <b-badge v-if="recipe.isFavoriteByUser" variant="danger" class="mx-1">❤️ מועדף</b-badge>
        </div>
      </div>
    </div>
  </router-link>
</template>

<script>
export default {
  name: "RecipePreview",
  props: {
    recipe: {
      type: Object,
      required: true
    }
  }
}
</script>

<style scoped>
.recipe-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}
.card-link {
  text-decoration: none;
  color: inherit;
}
.recipe-card:hover {
  transform: scale(1.02);
  transition: transform 0.2s ease-in-out;
}
</style>

