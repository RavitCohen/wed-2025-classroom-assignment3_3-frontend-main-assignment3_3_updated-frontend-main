<template>
  <div class="container">
    <div v-if="recipe">
      <!-- כותרת ותמונה -->
      <div class="recipe-header mt-3 mb-4 text-center">
        <h1>{{ recipe.title }}</h1>
        <img :src="recipe.image" class="center img-fluid rounded shadow" />
      </div>

      <!-- תגיות והמצב עבור המשתמש -->
      <div class="text-center mb-3">
        <b-badge v-if="recipe.vegan" variant="success" class="mx-1">🌱 טבעוני</b-badge>
        <b-badge v-if="recipe.vegetarian" variant="primary" class="mx-1">🥕 צמחוני</b-badge>
        <b-badge v-if="recipe.glutenFree" variant="warning" class="mx-1">🚫 ללא גלוטן</b-badge>

        <!-- תוויות למשתמש -->
        <b-badge v-if="recipe.isFavoriteByUser" variant="danger" class="mx-1">❤️ במועדפים</b-badge>
        <b-badge v-if="recipe.isWatched" variant="info" class="mx-1">👁️ נצפה לאחרונה</b-badge>
      </div>

      <div class="recipe-body">
        <div class="row">
          <!-- עמודת רכיבים -->
          <div class="col-md-6">
            <h4>🧂 מצרכים</h4>
            <p>ל{{ recipe.servings }} מנות | זמן הכנה: {{ recipe.readyInMinutes }} דקות</p>
            <ul>
              <li v-for="(r, index) in recipe.extendedIngredients" :key="index">
                {{ r.original }}
              </li>
            </ul>
          </div>

          <!-- עמודת הוראות -->
          <div class="col-md-6">
            <h4>📖 הוראות הכנה</h4>
            <div v-html="recipe.instructions"></div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="text-center my-5 text-muted">
      טוען מתכון...
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      recipe: null,
    };
  },
  async created() {
    try {
      let response = await this.axios.get(
        this.$root.store.server_domain + "/recipes/" + this.$route.params.recipeId
      );

      if (response.status !== 200) {
        this.$router.replace("/NotFound");
        return;
      }

      let {
        title,
        readyInMinutes,
        image,
        vegan,
        vegetarian,
        glutenFree,
        extendedIngredients,
        instructions,
        servings,
        isFavoriteByUser,
        isWatched,
      } = response.data;

      this.recipe = {
        title,
        readyInMinutes,
        image,
        vegan,
        vegetarian,
        glutenFree,
        extendedIngredients:
          typeof extendedIngredients === "string"
            ? JSON.parse(extendedIngredients)
            : extendedIngredients,
        instructions,
        servings,
        isFavoriteByUser,
        isWatched,
      };
    } catch (error) {
      console.error("Error loading recipe", error);
      this.$router.replace("/NotFound");
    }
  },
};
</script>

<style scoped>
.center {
  display: block;
  margin-left: auto;
  margin-right: auto;
  max-width: 100%;
}
.recipe-body {
  margin-top: 20px;
}
</style>
