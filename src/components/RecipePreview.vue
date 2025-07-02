<template>
  <div @click="handleClick" class="card-link">
    <div class="recipe-card-horizontal d-flex align-items-center">
      <img
        v-if="recipe.image"
        :src="recipe.image"
        class="recipe-image-horizontal"
        alt="תמונת מתכון"
      />

      <div class="recipe-details flex-grow-1 pe-3">
        <h5 class="card-title mb-2">{{ recipe.title }}</h5>
        <p class="card-text">⏱ זמן הכנה: {{ recipe.readyInMinutes }} דקות</p>

        <div class="badges d-flex flex-wrap mb-2">
          <b-badge v-if="recipe.vegetarian" variant="success" class="badge-item">🥕 צמחוני</b-badge>
          <b-badge v-if="recipe.vegan" variant="success" class="badge-item">🌱 טבעוני</b-badge>
          <b-badge v-if="recipe.glutenFree" variant="warning" class="badge-item">🚫 גלוטן</b-badge>
          <b-badge v-if="recipe.isWatched" variant="info" class="badge-item">👁️ נצפה</b-badge>
          <b-badge v-if="isFavorite" variant="danger" class="badge-item">❤️ מועדף</b-badge>
        </div>

        <div v-if="isLoggedIn">
          <b-button
            v-if="!isFavorite"
            variant="outline-danger"
            size="sm"
            @click.stop="markAsFavorite"
          >
            ❤️ הוסף למועדפים
          </b-button>
          <b-button
            v-else
            variant="outline-secondary"
            size="sm"
            @click.stop="removeFromFavorites"
          >
            ❌ הסר ממועדפים
          </b-button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import store from "@/store";
import { computed, ref } from 'vue';

export default {
  name: "RecipePreview",
  props: {
    recipe: { type: Object, required: true }
  },
  setup(props) {
    const isLoggedIn = computed(() => !!store.username.value);
    // ניהול מצב פנימי של מועדף
    const isFavorite = ref(props.recipe.isFavoriteByUser || false);
    return { isLoggedIn, isFavorite };
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
        this.isFavorite = true;
        this.$emit("update-favorite", this.recipe.id, true);
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
        this.isFavorite = false;
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
.recipe-card-horizontal {
  border: 1px solid #ddd;
  border-radius: 0.75rem;
  padding: 0.75rem;
  transition: box-shadow 0.2s, transform 0.2s;
  cursor: pointer;
  background: #fff;
  min-height: 150px; 
}

.recipe-card-horizontal:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.12);
  transform: translateY(-2px);
}

.recipe-details {
  display: flex;
  flex-direction: column;
  justify-content: space-between; 
}

.recipe-image-horizontal {
  width: 120px;
  height: 100px;
  object-fit: cover;
  border-radius: 0.5rem;
}

.card-title {
  font-weight: 700;       
  font-size: 1.15rem;    
  color: #222;           
}

.card-text {
  font-size: 0.95rem;
  color: #555;
}

.recipe-details b-button {
  font-weight: 600;
}

.badges {
  gap: 0.4rem;
  flex-wrap: wrap;
}

.badge-item {
  font-size: 0.85rem;   
}

@media (max-width: 576px) {
  .recipe-card-horizontal {
    flex-direction: column;
    align-items: flex-start;
  }
  .recipe-image-horizontal {
    width: 100%;
    height: 150px;
    margin-bottom: 0.5rem;
  }
  .recipe-details {
    width: 100%;
  }
}
</style>