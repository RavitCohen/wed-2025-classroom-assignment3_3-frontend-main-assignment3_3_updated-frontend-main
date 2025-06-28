<template>
  <div @click="handleClick" class="card-link">
    <div class="card h-100 recipe-card" :key="recipe.popularity">
      <img
        v-if="recipe.image"
        :src="recipe.image"
        class="card-img-top recipe-image"
        alt="Recipe image"
      />
      <div class="card-body text-center">
        <h5 class="card-title">{{ recipe.title }}</h5>
        <p class="card-text">{{ recipe.readyInMinutes }} דקות</p>

        <div class="badges my-2">
          <b-badge v-if="recipe.vegetarian" variant="success" class="mx-1">🥕 צמחוני</b-badge>
          <b-badge v-if="recipe.vegan" variant="success" class="mx-1">🌱 טבעוני</b-badge>
          <b-badge v-if="recipe.glutenFree" variant="warning" class="mx-1">🚫 גלוטן</b-badge>
          <b-badge v-if="recipe.isWatched" variant="info" class="mx-1">👁️ נצפה</b-badge>
          <b-badge v-if="recipe.isFavoriteByUser" variant="danger" class="mx-1">❤️ מועדף</b-badge>
        </div>

        <div class="d-flex justify-content-center mt-2">
          <button
            v-if="isLoggedIn && !recipe.isFavoriteByUser"
            @click.stop="markAsFavorite"
            class="btn btn-outline-danger mx-1"
          >
            ❤️הוספה למועדפים 
          </button>

          <b-button
            v-if="isLoggedIn && recipe.isFavoriteByUser"
            variant="outline-danger"
            size="sm"
            class="mt-2"
            @click.stop="removeFromFavorites">
            ❌ הסר ממועדפים
          </b-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import store from "@/store";
import { computed } from 'vue';

export default {
  name: "RecipePreview",
  props: {
    recipe: {
      type: Object,
      required: true
    }
  },
  setup() {
    const isLoggedIn = computed(() => !!store.username.value);
    return { isLoggedIn };
  },
  methods: {
    async handleClick() {
      try {
        await this.axios.post(
          `${this.$root.store.server_domain}/user/watch`,
          { recipeID: this.recipe.id },
          { withCredentials: true }
        );
      } catch (err) {
        console.error("שגיאה בסימון נצפה:", err);
      }
      this.$router.push({ name: "recipe", params: { recipeId: this.recipe.id } });
    },

    async markAsFavorite() {
      try {
        await this.axios.post(
          `${this.$root.store.server_domain}/user/favorites`,
          { recipeID: this.recipe.id },
          { withCredentials: true }
        );
        this.$emit("update-favorite", this.recipe.id, true);
        alert("❤️ נוסף למועדפים!");
      } catch (err) {
        console.error("שגיאה בסימון מועדף", err);
      }
    },

    async removeFromFavorites() {
      try {
        await this.axios.delete(
          `${this.$root.store.server_domain}/user/favorites/${this.recipe.id}`,
          { withCredentials: true }
        );
        this.$emit("update-favorite", this.recipe.id, false); 
        this.$emit("refresh"); 
      } catch (err) {
        console.error("שגיאה בהסרת מועדף:", err);
      }
    }
  }
};
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

