<template>
  <b-modal
    :model-value="show"
    @update:model-value="onUpdateShow"
    title="📝 יצירת מתכון חדש"
    hide-footer
    size="md"
    content-class="custom-modal-content"
    body-class="custom-modal-body"
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

      <b-form-group label="מאפייני תזונה">
        <div class="d-flex flex-wrap gap-3">
          <b-form-checkbox v-model="form.vegan">טבעוני</b-form-checkbox>
          <b-form-checkbox v-model="form.vegetarian">צמחוני</b-form-checkbox>
          <b-form-checkbox v-model="form.glutenFree">ללא גלוטן</b-form-checkbox>
        </div>
      </b-form-group>


      <b-form-group label="רשימת מרכיבים (extendedIngredients)">
        <b-form-textarea
          v-model="form.extendedIngredients"
          placeholder="דוגמה: 2 כוסות סוכר, 3 ביצים, 200 גרם חמאה"
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

      <div class="button-row">
        <b-button variant="secondary" @click="onUpdateShow(false)">
          ביטול
        </b-button>
        <b-button type="submit" variant="success">
          שמור מתכון
        </b-button>
      </div>

    </b-form>
  </b-modal>
</template>

<script>
import { ref, watch, nextTick } from "vue";
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


        const resetScroll = () => {
      nextTick(() => {
        const el = document.querySelector('.modal-body');
        if (el) {
          el.scrollTop = 0;
        }
      });
    };

    const onUpdateShow = (val) => {
      emit("update:show", val);
      if (!val) {
        resetScroll();
      }
    };


    const submitRecipe = async () => {
      try {
        const payload = {
          title: form.value.title || "",
          readyInMinutes: form.value.readyInMinutes || 0,
          image: form.value.image || "",
          vegan: !!form.value.vegan,
          vegetarian: !!form.value.vegetarian,
          glutenFree: !!form.value.glutenFree,
          extendedIngredients: form.value.extendedIngredients || "",
          instructions: form.value.instructions || "",
          servings: form.value.servings || 1
        };

        await axios.post("http://localhost:3000/user/recipes/", payload, {
          withCredentials: true,
        });
        emit("update:show", false);
        resetScroll();
        alert("המתכון נשמר בהצלחה");
        router.push({ name: "myRecipes" });
      } catch (err) {
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

<style >
/* הקטנת רוחב המודל */
.modal-dialog {
  max-width: 500px !important;
  margin: 15vh auto !important;
}

/* גלילה פנימית */
.modal-content .modal-body {
  max-height: 60vh !important;
  overflow-y: auto !important;
  padding-left: 1rem !important;
  padding-right: 1rem !important;
}

/* יישור ימין */
.modal-content .modal-body {
  direction: rtl !important;
  text-align: right !important;
}

/* עיצוב מסגרת */
.modal-content {
  border: 1px solid #ddd !important;
  border-radius: 0.75rem !important;
  box-shadow: 0 4px 20px rgba(0,0,0,0.2) !important;
}

.button-row {
  display: flex;
  justify-content: center; /* מרכז את הכפתורים */
  gap: 1rem;               /* מרווח ביניהם */
  margin-top: 1.5rem;      /* רווח מהשדות למעלה */
}


</style>
