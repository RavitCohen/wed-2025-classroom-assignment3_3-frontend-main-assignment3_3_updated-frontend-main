<template>
  <div class="container">
    <h1 class="title text-center mb-4">עמוד חיפוש מתכון</h1>

    <!-- טופס חיפוש -->
    <b-card class="p-4 mx-auto search-form mt-4">
      <b-form @submit.prevent="searchRecipes">
        <!-- שורה ראשונה: שם מתכון ו-Limit -->
        <b-row class="mb-3">
          <b-col md="8">
            <b-form-group label="שם מתכון" label-align="center">
              <b-form-input v-model="query.title" placeholder="חפש לפי שם..." />
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="Limit (מספר תוצאות)" label-align="center">
              <b-form-select v-model="query.limit" :options="limitOptions" />
            </b-form-group>
          </b-col>
        </b-row>

        <!-- שורה שניה: Cuisine, Diet, Intolerance -->
        <b-row class="mb-3">
          <b-col md="4">
            <b-form-group label="Cuisine" label-align="center">
              <b-form-select v-model="query.cuisine" :options="cuisineOptions" />
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="Diet" label-align="center">
              <b-form-select v-model="query.diet" :options="dietOptions" />
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="Intolerance" label-align="center">
              <b-form-select v-model="query.intolerance" :options="intoleranceOptions" />
            </b-form-group>
          </b-col>
        </b-row>

        <!-- כפתור -->
        <b-row class="text-center">
          <b-col>
            <b-button type="submit" variant="primary" class="search-button">
              חפש
            </b-button>
          </b-col>
        </b-row>
      </b-form>
    </b-card>

    <!-- תוצאות -->
    <div v-if="recipes && recipes.length > 0" class="mt-4">
      <h3 class="text-center mb-3">תוצאות החיפוש</h3>
      <RecipePreviewList :recipes="recipes" @refresh="searchRecipes" />
    </div>

    <div v-else-if="searched && recipes && recipes.length === 0" class="text-center mt-3">
      <p class="text-danger">לא נמצאו תוצאות</p>
    </div>
  </div>
</template>

<script>
import RecipePreviewList from "@/components/RecipePreviewList.vue";

export default {
  name: 'SearchPage',
  components: { RecipePreviewList },
  data() {
    return {
      limitOptions: [
        { value: 5, text: '5' },
        { value: 10, text: '10' },
        { value: 15, text: '15' }
      ],
      recipes: [],
      searched: false,
      query: {
        title: '',
        cuisine: '',
        diet: '',
        intolerance: '',
        limit: 5
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
      console.log("searchRecipes invoked");
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
  max-width: 700px;
  border: 1px solid #ddd;
  border-radius: 0.75rem;
  background-color: #fafafa;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}
.search-button {
  width: 100%;
  font-size: 1.1rem;
}
</style>
