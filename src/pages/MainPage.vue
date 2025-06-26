// MainPage.vue - עמוד מתכונים ראשי
<template>
  <div class="container">
    <h1 class="title text-center my-4">עמוד מתכונים ראשי</h1>

    <BRow>
      <!-- עמודה שמאלית: מתכונים רנדומליים -->
      <BCol md="8">
        <RecipePreviewList title="Random Recipes" class="RandomRecipes center" :recipes="randomRecipes" />
        <div class="text-center mt-3">
          <BButton variant="info" @click="loadMoreRandom">טען עוד</BButton>
        </div>
      </BCol>

      <!-- עמודה ימנית: אחרונים שצפה / התחברות -->
      <BCol md="4">
        <div v-if="!store.username" class="text-center mt-4">
          <BContainer class="text-center my-5">
            <div class="custom-alert">
              <h5 class="mb-2">ברוך הבא לאתר מתכוני סבתא 👵</h5>
              <p class="mb-0">כדי לצפות במתכונים שצפית לאחרונה – התחבר או הירשם</p>
            </div>
            <BRow class="justify-content-center mt-3">
              <BCol cols="auto">
                <router-link :to="{ name: 'login' }">
                  <BButton variant="primary">התחברות</BButton>
                </router-link>
              </BCol>
              <BCol cols="auto">
                <router-link :to="{ name: 'register' }">
                  <BButton variant="success">הרשמה</BButton>
                </router-link>
              </BCol>
            </BRow>
          </BContainer>
        </div>

        <RecipePreviewList
          v-else
          title="Last Viewed Recipes"
          :class="{
            RandomRecipes: true,
            blur: !store.username,
            center: true
          }"
          :recipes="lastViewedRecipes"
          disabled
        />
      </BCol>
    </BRow>
  </div>
</template>

<script>
import { getCurrentInstance, ref, onMounted } from 'vue';
import RecipePreviewList from "../components/RecipePreviewList.vue";
import axios from 'axios';

export default {
  components: {
    RecipePreviewList
  },
  setup() {
    const internalInstance = getCurrentInstance();
    const store = internalInstance.appContext.config.globalProperties.store;

    const randomRecipes = ref([]);
    const lastViewedRecipes = ref([]);

    const loadMoreRandom = async () => {
      try {
        const res = await axios.get('http://localhost:3000/recipes');
        randomRecipes.value = res.data;
      } catch (err) {
        console.error('שגיאה בטעינת מתכונים רנדומליים:', err);
      }
    };

    const loadLastViewed = async () => {
      try {
        const res = await axios.get('http://localhost:3000/recipes/last-viewed', {
          headers: {
            Authorization: `Bearer ${store.token}`
          }
        });
        lastViewedRecipes.value = res.data;
      } catch (err) {
        console.error('שגיאה בטעינת צפיות אחרונות:', err);
      }
    };

    onMounted(() => {
      loadMoreRandom();
      if (store.username) {
        loadLastViewed();
      }
    });

    return { store, loadMoreRandom, randomRecipes, lastViewedRecipes };
  }
};
</script>

<style lang="scss">
.RandomRecipes {
  margin: 10px 0 10px;
}
.blur {
  -webkit-filter: blur(5px);
  filter: blur(2px);
}
::v-deep .blur .recipe-preview {
  pointer-events: none;
  cursor: default;
}
</style>
