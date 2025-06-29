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

      <b-form-group label="זמן הכנה (בדקות)">
        <b-form-input v-model.number="form.readyInMinutes" type="number" min="1" required />
      </b-form-group>

      <b-form-group label="תמונה (קישור URL)">
        <b-form-input v-model="form.image" type="url" />
      </b-form-group>

      <b-form-group label="האם טבעוני?">
        <b-form-checkbox v-model="form.vegan">כן</b-form-checkbox>
      </b-form-group>

      <b-form-group label="האם צמחוני?">
        <b-form-checkbox v-model="form.vegetarian">כן</b-form-checkbox>
      </b-form-group>

      <b-form-group label="ללא גלוטן?">
        <b-form-checkbox v-model="form.glutenFree">כן</b-form-checkbox>
      </b-form-group>

      <b-form-group label="רשימת מרכיבים (extendedIngredients)">
        <b-form-textarea
          v-model="form.extendedIngredients"
          placeholder="הפרד בין רכיבים עם פסיקים (,) לדוגמה: קמח, ביצים, סוכר"
          rows="3"
          required
        />
      </b-form-group>

      <b-form-group label="הוראות הכנה">
        <b-form-textarea v-model="form.instructions" rows="4" required />
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
    readyInMinutes: 1,
    image: "",
    vegan: "",
    vegetarian: "",
    glutenFree: "",
    extendedIngredients: "",
    instructions: "",
    servings: 1,
    });

    const onUpdateShow = (val) => {
      emit("update:show", val);
    };

    const submitRecipe = async () => {
      try {
        console.log(`User input: ${payload}`);
        const payload = {
          ...form.value,
          extendedIngredients: form.value.extendedIngredients
            .split(",")
            .map((line, index) => {
              const parts = line.trim().split(" ");
              const amount = parseFloat(parts[0]) || 1;
              const unit = isNaN(parseFloat(parts[1])) ? parts[1] : '';
              const name = parts.slice(unit ? 2 : 1).join(" ");

              return {
                id: index + 1,
                amount,
                unit,
                name,
                original: line.trim()
              };
            })
          }
        await axios.post("http://localhost:3000/user/recipes/", payload, {
          withCredentials: true,
        });
        console.log("Create recipes invoked successfully on Node.js");
        emit("update:show", false);
        alert("המתכון נשמר בהצלחה", "", "success");

        // redirect to MyRecipes...
        router.push({ name: "myRecipes" });
      } catch (err) {
        console.error("שגיאה ביצירת מתכון:", err);
        console.error("שגיאה ביצירת מתכון:", err?.response?.data || err.message || err);

      }
    };

    watch(
      () => props.show,
      (newVal) => {
        if (!newVal) {
          form.value = {
            title: "",
            readyInMinutes: 1,
            image: "",
            vegan: "",
            vegetarian: "",
            glutenFree: "",
            extendedIngredients: "",
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
