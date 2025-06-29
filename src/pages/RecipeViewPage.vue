<template>
  <div class="container">
    <div v-if="recipe" class="recipe-card animate-fade-in">
      <!-- כותרת -->
      <h1 class="recipe-title">{{ recipe.title }}</h1>

      <!-- תמונה -->
      <img :src="recipe.image" class="recipe-image shadow" />

      <!-- תגיות -->
      <div class="tags text-center mt-2 mb-3">
        <b-badge v-if="recipe.vegan" variant="success" class="mx-1">🌱 טבעוני</b-badge>
        <b-badge v-if="recipe.vegetarian" variant="primary" class="mx-1">🥕 צמחוני</b-badge>
        <b-badge v-if="recipe.glutenFree" variant="warning" class="mx-1">🚫 ללא גלוטן</b-badge>
        <b-badge v-if="recipe.isFavoriteByUser" variant="danger" class="mx-1">❤️ במועדפים</b-badge>
        <b-badge v-if="recipe.isWatched" variant="info" class="mx-1">👁️ נצפה לאחרונה</b-badge>
      </div>

      <div class="row recipe-body">
        <!-- מצרכים -->
        <div class="col-md-6 ingredients-box">
          <h4 class="section-title">🧂 מצרכים</h4>
          <p class="info-text">
            ל{{ recipe.servings }} מנות | זמן הכנה: {{ recipe.readyInMinutes }} דקות
          </p>
          <ul>
            <li v-for="(r, index) in recipe.extendedIngredients" :key="index">
              {{ r.original }}
            </li>
          </ul>
        </div>

        <!-- הוראות -->
        <div class="col-md-6 instructions-box">
          <h4 class="section-title">📖 הוראות הכנה</h4>
          <div v-html="recipe.instructions"></div>
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
.recipe-card {
  background-color: #ffffff;
  border: 1px solid #ddd;
  border-radius: 0.75rem;
  padding: 1.5rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  max-width: 1000px;
  margin: auto;
}

.recipe-title {
  font-size: 2rem;
  font-weight: 700;
  text-align: center;
  margin-bottom: 1rem;
}

.recipe-image {
  display: block;
  margin: 0 auto 1rem auto;
  max-width: 100%;
  border-radius: 0.5rem;
}

.tags {
  font-size: 1rem;
}

.section-title {
  font-size: 1.3rem;
  font-weight: 600;
  border-bottom: 1px solid #ddd;
  padding-bottom: 0.3rem;
  margin-bottom: 0.75rem;
}

.info-text {
  font-size: 0.95rem;
  color: #555;
  margin-bottom: 0.75rem;
}

.ingredients-box,
.instructions-box {
  background-color: #f9f9f9;
  border-radius: 0.5rem;
  padding: 1rem;
  margin-bottom: 1rem;
}

.animate-fade-in {
  animation: fadeIn 0.6s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

</style>
