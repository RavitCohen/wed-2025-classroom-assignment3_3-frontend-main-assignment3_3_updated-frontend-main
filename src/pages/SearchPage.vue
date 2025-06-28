<template>
  <div class="container">
    <h1 class="title text-center mb-4">עמוד חיפוש מתכון</h1>

    <!-- טופס חיפוש -->
    <b-card class="p-4 mx-auto search-form">
      <b-form @submit.prevent="searchRecipes">
        <b-row class="mb-3">
          <b-col cols="12">
            <b-form-group label="שם מתכון" label-cols-sm="3">
              <b-form-input v-model="query.title" placeholder="חפש לפי שם..." />
            </b-form-group>
          </b-col>
        </b-row>

        <b-row class="mb-3">
          <b-col md="4">
            <b-form-group label="Cuisine">
              <b-form-select v-model="query.cuisine" :options="cuisineOptions" />
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="Diet">
              <b-form-select v-model="query.diet" :options="dietOptions" />
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="Intolerance">
              <b-form-select v-model="query.intolerance" :options="intoleranceOptions" />
            </b-form-group>
          </b-col>
        </b-row>

        <b-row class="mb-3">
          <b-col md="6">
            <b-form-group label="Limit (מספר תוצאות)">
              <b-form-input
                type="number"
                v-model="query.limit"
                :min="1"
                :max="50"
                placeholder="לדוג׳ 5"
              />
            </b-form-group>
          </b-col>
        </b-row>
        <b-row class="text-center">
          <b-col>
            <b-button type="submit" variant="primary">חפש</b-button>
          </b-col>
        </b-row>
      </b-form>
    </b-card>

    <!-- תוצאות חיפוש -->
    <div v-if="recipes && recipes.length > 0" class="mt-4">
      <h3 class="text-center">תוצאות החיפוש</h3>
      <div class="row">
        <div class="col-md-4 mb-3" v-for="r in recipes" :key="r.id">
          <RecipePreview :recipe="r" />
        </div>
      </div>
    </div>

    <div v-else-if="searched && recipes && recipes.length === 0" class="text-center mt-3">
      <p class="text-danger">לא נמצאו תוצאות</p>
    </div>
  </div>
</template>

<script>
import RecipePreview from '@/components/RecipePreview.vue';

export default {
  name: 'SearchPage',
  components: {
    RecipePreview
  },
  data() {
    return {
      recipes: [],
      searched: false,
      query: {
        query: '',
        cuisine: '',
        diet: '',
        intolerance: '',
        limit: ''
      },
      cuisineOptions: [
        '', 'African', 'Asian', 'American', 'British', 'Cajun', 'Caribbean', 'Chinese',
        'Eastern European', 'European', 'French', 'German', 'Greek', 'Indian', 'Irish',
        'Italian', 'Japanese', 'Jewish', 'Korean', 'Latin American', 'Mediterranean',
        'Mexican', 'Middle Eastern', 'Nordic', 'Southern', 'Spanish', 'Thai', 'Vietnamese'
      ],
      dietOptions: [
        '', 'Gluten Free', 'Ketogenic', 'Vegetarian', 'Lacto-Vegetarian',
        'Ovo-Vegetarian', 'Vegan', 'Pescetarian', 'Paleo', 'Primal', 'Low FODMAP', 'Whole30'
      ],
      intoleranceOptions: [
        '', 'Dairy', 'Egg', 'Gluten', 'Grain', 'Peanut', 'Seafood', 'Sesame',
        'Shellfish', 'Soy', 'Sulfite', 'Tree Nut', 'Wheat'
      ]
    };
  },
  methods: {
    async searchRecipes() {
      console.log("searchRecipes invoked")
      try {
        const params = {
          query: this.query.title,
          cuisine: this.query.cuisine,
          diet: this.query.diet,
          intolerance: this.query.intolerance,
          limit: this.query.limit
        };
        const response = await this.axios.get(this.$root.store.server_domain + '/recipes/search', { params });
console.log("Recipes:", response.data);
response.data.forEach((recipe, i) => {
  console.log(`🔸 Recipe ${i + 1}:`, recipe);
});
    this.recipes = response.data;
  } catch (error) {
    console.error('שגיאה בביצוע חיפוש:', error);
  } finally {
    this.searched = true;
  }
  }
  }
};
</script>

<style scoped>
.container {
  padding-top: 2rem;
  padding-bottom: 2rem;
  max-width: 900px;
}
.search-form {
  max-width: 800px;
}
</style>
