<template>
  <div class="d-flex flex-wrap justify-content-center mt-2 mb-3" v-if="tags.length">
    <RecipeTag v-for="t in tags" :key="t" :type="t" />
  </div>
</template>

<script>
import RecipeTag from "./RecipeTag.vue";

export default {
  name: "RecipeTagList",
  components: { RecipeTag },
  props: {
    recipe: { type: Object, required: true }
  },
  computed: {
    tags() {
      const out = [];
      if (this.recipe?.vegan) out.push("vegan");
      if (this.recipe?.vegetarian) out.push("vegetarian");
      if (this.recipe?.glutenFree) out.push("glutenFree");
      if (this.recipe?.isWatched) out.push("watched");
      if (this.recipe?.isFavoriteByUser) out.push("favorite");
      return out;
    }
  }
};
</script>
