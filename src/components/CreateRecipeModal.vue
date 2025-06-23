<template>
  <b-modal
    :model-value="show"
    @update:model-value="onUpdateShow"
    title="📝 יצירת מתכון חדש"
    hide-footer
  >
    <b-form @submit.prevent="submitRecipe">
      <b-form-group label="שם המתכון">
        <b-form-input v-model="form.title" required />
      </b-form-group>

      <b-form-group label="תיאור">
        <b-form-textarea v-model="form.description" rows="3" />
      </b-form-group>

      <b-form-group label="תמונה (קישור URL)">
        <b-form-input v-model="form.image" type="url" />
      </b-form-group>

      <b-form-group label="רשימת מרכיבים">
        <b-form-textarea
          v-model="form.ingredients"
          placeholder="הפרד בין רכיבים עם פסיקים (,)"
          rows="3"
        />
      </b-form-group>

      <b-form-group label="הוראות הכנה">
        <b-form-textarea v-model="form.instructions" rows="4" />
      </b-form-group>

      <b-form-group label="מספר מנות">
        <b-form-input
          v-model.number="form.servings"
          type="number"
          min="1"
          required
        />
      </b-form-group>

      <div class="text-end">
        <b-button variant="secondary" class="me-2" @click="onUpdateShow(false)">
          ביטול
        </b-button>
        <b-button type="submit" variant="success">שמור מתכון</b-button>
      </div>
    </b-form>
  </b-modal>
</template>

<script>
import { ref, watch } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import store from "@/store";

export default {
  name: "CreateRecipeModal",
  props: {
    show: Boolean,
  },
  emits: ["update:show"],
  setup(props, { emit }) {
    const router = useRouter();

    const form = ref({
      title: "",
      description: "",
      image: "",
      ingredients: "",
      instructions: "",
      servings: 1,
    });

    const onUpdateShow = (val) => {
      emit("update:show", val);
    };

    const submitRecipe = async () => {
      try {
        const payload = {
          ...form.value,
          ingredients: form.value.ingredients
            .split(",")
            .map((i) => i.trim()),
        };
        await axios.post("http://localhost:3000/recipes/create", payload, {
          headers: { Authorization: `Bearer ${store.token}` },
        });

        emit("update:show", false);
        window.toast("המתכון נשמר בהצלחה", "", "success");

        // ניווט לעמוד המתכונים של המשתמש
        router.push({ name: "myRecipes" });
      } catch (err) {
        console.error("שגיאה ביצירת מתכון:", err);
        window.toast("שגיאה בשמירה", "", "danger");
      }
    };

    watch(
      () => props.show,
      (newVal) => {
        if (!newVal) {
          form.value = {
            title: "",
            description: "",
            image: "",
            ingredients: "",
            instructions: "",
            servings: 1,
          };
        }
      }
    );

    return { form, submitRecipe, onUpdateShow };
  },
};
</script>

<style scoped>
.me-2 {
  margin-inline-end: 0.5rem;
}
</style>
