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
      <!-- שם המתכון -->
      <b-form-group label="שם המתכון">
        <b-form-input
          v-model="form.title"
          :state="getValidationState(v$.title)"
          @blur="v$.title.$touch"
        />
        <b-form-invalid-feedback v-if="v$.title.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- זמן הכנה -->
      <b-form-group label="זמן הכנה (בדקות)">
        <b-form-input
          v-model.number="form.readyInMinutes"
          type="number"
          min="1"
          :state="getValidationState(v$.readyInMinutes)"
          @blur="v$.readyInMinutes.$touch"
        />
        <b-form-invalid-feedback v-if="v$.readyInMinutes.$error">
          יש להזין מספר גדול מ-0.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- תמונה -->
      <b-form-group label="תמונה (קישור URL)">
        <b-form-input
          v-model="form.image"
          type="text"
          :state="getValidationState(v$.image)"
          @blur="v$.image.$touch"
        />
        <b-form-invalid-feedback v-if="v$.image.$error">
          יש להזין כתובת URL תקינה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- מאפייני תזונה -->
      <b-form-group label="מאפייני תזונה">
        <div class="d-flex flex-wrap gap-3">
          <b-form-checkbox v-model="form.vegan">טבעוני</b-form-checkbox>
          <b-form-checkbox v-model="form.vegetarian">צמחוני</b-form-checkbox>
          <b-form-checkbox v-model="form.glutenFree">ללא גלוטן</b-form-checkbox>
        </div>
      </b-form-group>

      <!-- מצרכים -->
      <b-form-group label="רשימת מצרכים">
        <b-form-textarea
          v-model="form.extendedIngredients"
          rows="3"
          :state="getValidationState(v$.extendedIngredients)"
          @blur="v$.extendedIngredients.$touch"
          placeholder="יש להפריד באמצעות פסיק (,)"
        />
        <b-form-invalid-feedback v-if="v$.extendedIngredients.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- הוראות הכנה -->
      <b-form-group label="הוראות הכנה">
        <b-form-textarea
          v-model="form.instructions"
          rows="4"
          :state="getValidationState(v$.instructions)"
          @blur="v$.instructions.$touch"
        />
        <b-form-invalid-feedback v-if="v$.instructions.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- מספר מנות -->
      <b-form-group label="מספר מנות">
        <b-form-input
          v-model.number="form.servings"
          type="number"
          min="1"
          :state="getValidationState(v$.servings)"
          @blur="v$.servings.$touch"
        />
        <b-form-invalid-feedback v-if="v$.servings.$error">
          יש להזין מספר גדול מ-0.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- הודעות הצלחה/שגיאה -->
      <div v-if="state.successMessage" class="alert alert-success mt-3 text-center">
        {{ state.successMessage }}
      </div>

      <div v-if="state.submitError" class="alert alert-danger mt-3 text-center">
        {{ state.submitError }}
      </div>

      <div class="button-row">
        <b-button variant="secondary" @click="onUpdateShow(false)">ביטול</b-button>
        <b-button type="submit" variant="success">שמור מתכון</b-button>
      </div>
    </b-form>
  </b-modal>
</template>

<script>
import { reactive, watch } from "vue";
import { useVuelidate } from "@vuelidate/core";
import { required, minValue, url } from "@vuelidate/validators";
import axios from "axios";
import { useRouter } from "vue-router";

export default {
  name: "CreateRecipeModal",
  props: {
    show: Boolean,
  },
  emits: ["update:show"],
  setup(props, { emit }) {
    const router = useRouter();

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

    const rules = {
      title: { required },
      readyInMinutes: { required, minValue: minValue(1) },
      image: { required, url },
      extendedIngredients: { required },
      instructions: { required },
      servings: { required, minValue: minValue(1) },
    };

    const v$ = useVuelidate(rules, form);

    const state = reactive({
      successMessage: "",
      submitError: "",
    });

    const getValidationState = (field) => {
      return field.$dirty ? !field.$invalid : null;
    };

    const submitRecipe = async () => {
      const valid = await v$.value.$validate();
      if (!valid) return;

      try {
        state.successMessage = "";
        state.submitError = "";

        const payload = { ...form };

        await axios.post("http://localhost:3000/user/recipes/", payload, {
          withCredentials: true,
        });

        state.successMessage = "המתכון נשמר בהצלחה!";
        setTimeout(() => {
          emit("update:show", false);
          router.push({ name: "myRecipes" });
        }, 3000);
      } catch (err) {
        console.error("שגיאה ביצירת מתכון:", err?.response?.data || err.message || err);
        state.submitError = "אירעה שגיאה בשמירת המתכון.";
      }
    };

    const onUpdateShow = (val) => {
      emit("update:show", val);
      if (!val) {
        v$.value.$reset();
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
    };

    watch(() => props.show, (newVal) => {
      if (!newVal) {
        v$.value.$reset();
      }
    });

    return { form, state, submitRecipe, v$, getValidationState, onUpdateShow };
  },
};
</script>


<style >
.modal-dialog {
  max-width: 500px !important;
  margin: 15vh auto !important;
}

.modal-header {
  background: linear-gradient(90deg, #0d6efd, #6610f2);
  color: white;
  font-weight: 600;
  border-top-left-radius: 0.75rem;
  border-top-right-radius: 0.75rem;
}

.modal-content .modal-body {
  max-height: 65vh !important;
  overflow-y: auto !important;
  padding-left: 1rem !important;
  padding-right: 1rem !important;
}

.modal-content .modal-body {
  direction: rtl !important;
  text-align: right !important;
}

.modal-content {
  border: 1px solid #9a9595 !important;
  border-radius: 0.7rgb(71, 42, 204)mportant;
  box-shadow: 0 4px 20px rgba(0,0,0,0.4) !important;
}

.button-row {
  display: flex;
  justify-content: center; 
  gap: 1rem;              
  margin-top: 1.5rem;    
}


</style>