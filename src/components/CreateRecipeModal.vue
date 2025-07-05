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
        <b-form-input v-model="form.image" type="url" required/>
      </b-form-group>

      <b-form-group label="מאפייני תזונה">
        <div class="d-flex flex-wrap gap-3">
          <b-form-checkbox v-model="form.vegan">טבעוני</b-form-checkbox>
          <b-form-checkbox v-model="form.vegetarian">צמחוני</b-form-checkbox>
          <b-form-checkbox v-model="form.glutenFree">ללא גלוטן</b-form-checkbox>
        </div>
      </b-form-group>


      <b-form-group label="רשימת מצרכים">
        <b-form-textarea
          v-model="form.extendedIngredients"
          :placeholder="'יש להפריד באמצעות פסיק (,)\nדוגמה: 2 כוסות סוכר, 3 ביצים, 200 גרם חמאה'"
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

      <div
        v-if="state.successMessage"
        class="alert alert-success mt-3 text-center"
        role="alert"
      >
        {{ state.successMessage }}
      </div>

      <div
        v-if="state.submitError"
        class="alert alert-danger mt-3 text-center"
        role="alert"
      >
        {{ state.submitError }}
      </div>

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
import { reactive, watch, nextTick } from "vue";
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

    const state = reactive({
      successMessage: "",
      submitError: ""
    });

    const form = reactive({
      title: "",
      readyInMinutes: 1,
      image: "",
      vegan: false,
      vegetarian: false,
      glutenFree: false,
      extendedIngredients: "",
      instructions: "",
      servings: 1,
    });

    const resetScroll = () => {
      nextTick(() => {
        const el = document.querySelector(".modal-body");
        if (el) {
          el.scrollTop = 0;
        }
      });
    };

    const onUpdateShow = (val) => {
      emit("update:show", val);
      if (!val) {
        resetScroll();
        state.successMessage = "";
        state.submitError = "";
      }
    };

    const submitRecipe = async () => {
      try {
        state.successMessage = "";
        state.submitError = "";

        const payload = {
          title: form.title || "",
          readyInMinutes: form.readyInMinutes || 0,
          image: form.image || "",
          vegan: !!form.vegan,
          vegetarian: !!form.vegetarian,
          glutenFree: !!form.glutenFree,
          extendedIngredients: form.extendedIngredients || "",
          instructions: form.instructions || "",
          servings: form.servings || 1
        };

        await axios.post("http://localhost:3000/user/recipes/", payload, {
          withCredentials: true,
        });

        state.successMessage = "🎉 המתכון נשמר בהצלחה!";
        state.submitError = "";

        setTimeout(() => {
          emit("update:show", false);
          resetScroll();
          router.push({ name: "myRecipes" });
        }, 3000);

      } catch (err) {
        console.error("שגיאה ביצירת מתכון:", err?.response?.data || err.message || err);
        state.submitError = "אירעה שגיאה בשמירת המתכון.";
        state.successMessage = "";
      }
    };

    watch(
      () => props.show,
      (newVal) => {
        if (!newVal) {
          // איפוס כל הערכים
          Object.assign(form, {
            title: "",
            readyInMinutes: 1,
            image: "",
            vegan: false,
            vegetarian: false,
            glutenFree: false,
            extendedIngredients: "",
            instructions: "",
            servings: 1,
          });
          state.successMessage = "";
          state.submitError = "";
        }
      }
    );

    return { form, submitRecipe, onUpdateShow, state };
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
  max-height: 65vh !important;
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
  border: 1px solid #9a9595 !important;
  border-radius: 0.75rem !important;
  box-shadow: 0 4px 20px rgba(0,0,0,0.4) !important;
}

.button-row {
  display: flex;
  justify-content: center; /* מרכז את הכפתורים */
  gap: 1rem;               /* מרווח ביניהם */
  margin-top: 1.5rem;      /* רווח מהשדות למעלה */
}


</style>
