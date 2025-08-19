<template>
  <b-badge :variant="config.variant || null" :class="['mx-1', config.className]">
    <span v-if="config.icon" aria-hidden="true">{{ config.icon }}</span>
    {{ config.label }}
  </b-badge>
</template>

<script>
const TAGS = {
  vegan:       { label: "טבעוני",    icon: "🌱", variant: "success" },
  vegetarian:  { label: "צמחוני",    icon: "🥕", variant: "success" }, 
  glutenFree:  { label: "ללא גלוטן", icon: "🚫", variant: "warning" },
  watched:     { label: "נצפה",       icon: "👁️", variant: "info" },
  favorite:    { label: "מועדף",     icon: "❤️", variant: "danger" },
};

export default {
  name: "RecipeTag",
  props: {
    type: { type: String, required: true }
  },
  computed: {
    config() {
      return TAGS[this.type] ?? { label: this.type, icon: "", variant: "secondary" };
    }
  }
};
</script>

<style scoped>

.vegetarian-badge {
  background-color: #7ddc7d;
  color: #003300;
}
</style>
