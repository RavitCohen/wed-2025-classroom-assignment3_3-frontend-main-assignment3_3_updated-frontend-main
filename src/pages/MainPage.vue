<template>
  <div class="container py-4">
    <h1 class="title text-center mb-4">עמוד מתכונים ראשי</h1>

    <div class="recipes-wrapper">
      <!-- עמודה ימין -->
      <div class="recipes-section">
        <h3 class="section-title">המתכונים שצפית לאחרונה</h3>
        <div v-if="!isLoggedIn">
          <h4 class="mb-3 text-center">👵 ברוך הבא לאתר מתכוני סבתא</h4>
          <p class="text-center">כדי לצפות במתכונים שצפית לאחרונה, התחבר או הירשם:</p>
          <LoginPage />
        </div>
        <div v-else class="recipes-list-vertical">
          <RecipePreview
            v-for="r in lastViewedRecipes"
            :key="r.id"
            :recipe="r"
          />
        </div>
      </div>

      <!-- עמודה שמאל -->
      <div class="recipes-section">
        <h3 class="section-title">מתכונים מומלצים</h3>
        <div class="recipes-list-vertical">
          <RecipePreview
            v-for="r in randomRecipes"
            :key="r.id"
            :recipe="r"
          />
        </div>
        <div class="text-center mt-3">
          <b-button variant="info" @click="loadMoreRandom">טען עוד מתכונים</b-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import LoginPage from "./LoginPage.vue";
import RecipePreview from "../components/RecipePreview.vue";
import axios from 'axios';
import store from "@/store";

export default {
  components: {
    RecipePreview,
    LoginPage
  },
  setup() {
    const isLoggedIn = computed(() => !!store.username.value);
    const randomRecipes = ref([]);
    const lastViewedRecipes = ref([]);

    const loadMoreRandom = async () => {
      try {
        const res = await axios.get('http://localhost:3000/recipes', {
          withCredentials: true,
        });
        console.log("Random recipes:", res.data);
        randomRecipes.value = res.data;
      } catch (err) {
        console.error('שגיאה בטעינת מתכונים רנדומליים:', err);
      }
    };

    const loadLastViewed = async () => {
      try {
        const res = await axios.get('http://localhost:3000/user/watch', {
          withCredentials: true,
        });
        lastViewedRecipes.value = res.data;
      } catch (err) {
        console.error('שגיאה בטעינת צפיות אחרונות:', err);
      }
    };

    onMounted(() => {
      loadMoreRandom();
      if (store.username.value) {
        loadLastViewed();
      }
    });

    return { store, loadMoreRandom, randomRecipes, lastViewedRecipes, isLoggedIn };
  }
};
</script>

<style scoped>
.recipes-wrapper {
  display: flex;
  gap: 1rem;
  flex-wrap: nowrap;
  justify-content: center;
}

.recipes-section {
  flex: 0 0 48%;
  border: 1px solid #ddd;
  border-radius: 0.5rem;
  padding: 0.75rem;
  background-color: #fafafa;
  min-width: 300px;
  display: flex;
  flex-direction: column;
}

.section-title {
  text-align: center;
  font-weight: 600;
  font-size: 1.2rem;
  margin-bottom: 0.8rem;
  border-bottom: 2px solid #007bff;
  padding-bottom: 0.3rem;
}

.recipes-list-vertical {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

@media (max-width: 992px) {
  .recipes-wrapper {
    flex-direction: column;
  }
  .recipes-section {
    flex: 1 1 100%;
  }
}
</style>
 