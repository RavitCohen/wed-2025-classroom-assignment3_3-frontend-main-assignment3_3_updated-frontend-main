<template>
  <div class="container mt-4" style="max-width: 500px;">
    <h2 class="mb-4">הרשמה</h2>
    <b-form @submit.prevent="handleRegister">

      <!-- שם משתמש -->
      <b-form-group label="שם משתמש">
        <b-form-input
          v-model="state.username"
          :state="validations.username.$dirty ? !validations.username.$invalid : null"
          @input="validations.username.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.username.$error">
          <template v-for="error in validations.username.$errors" :key="error.$uid">
            <div>{{ error.$message }}</div>
          </template>
        </b-form-invalid-feedback>


      </b-form-group>

      <!-- שם פרטי -->
      <b-form-group label="שם פרטי">
        <b-form-input
          v-model="state.firstName"
          :state="validations.firstName.$dirty ? !validations.firstName.$invalid : null"
          @input="validations.firstName.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.firstName.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- שם משפחה -->
      <b-form-group label="שם משפחה">
        <b-form-input
          v-model="state.lastName"
          :state="validations.lastName.$dirty ? !validations.lastName.$invalid : null"
          @input="validations.lastName.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.lastName.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- אימייל -->
      <b-form-group label="אימייל">
      <b-form-input
        type="email"
        v-model="state.email"
        :state="validations.email.$dirty ? !validations.email.$invalid : null"
        @input="validations.email.$touch()"
      />
      <b-form-invalid-feedback v-if="validations.email.$error">
        <template v-for="error in validations.email.$errors" :key="error.$uid">
          <div>{{ error.$message }}</div>
        </template>
      </b-form-invalid-feedback>
    </b-form-group>


      <!-- מדינה -->
      <b-form-group label="מדינה">
        <b-form-select
          v-model="state.country"
          :options="state.countries"
          :state="validations.country.$dirty ? !validations.country.$invalid : null"
          @change="validations.country.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.country.$error">
          חובה לבחור מדינה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- סיסמה -->
      <b-form-group label="סיסמה">
        <b-form-input
          :type="showPassword ? 'text' : 'password'"
          v-model="state.password"
          :state="validations.password.$dirty ? !validations.password.$invalid : null"
          @input="validations.password.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.password.$error">
          <template v-for="error in validations.password.$errors" :key="error.$uid">
            <div>{{ error.$message }}</div>
          </template>
        </b-form-invalid-feedback>

      </b-form-group>

      <!-- אימות סיסמה -->
      <b-form-group label="אימות סיסמה">
        <b-form-input
          :type="showConfirmPassword ? 'text' : 'password'"
          v-model="state.confirmPassword"
          :state="validations.confirmPassword.$dirty ? !validations.confirmPassword.$invalid : null"
          @input="validations.confirmPassword.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.confirmPassword.$error">
          <template v-for="error in validations.confirmPassword.$errors" :key="error.$uid">
            <div>{{ error.$message }}</div>
          </template>
        </b-form-invalid-feedback>


      </b-form-group>

      <b-button type="submit" variant="success" class="w-100">הרשמה</b-button>

      <!-- הודעת שגיאה/הצלחה כללית -->
      <div v-if="state.submitError" class="mt-3 text-danger font-weight-bold">
        {{ state.submitError }}
      </div>
      <div v-if="state.successMessage" class="mt-3 text-success font-weight-bold">
        {{ state.successMessage }}
      </div>

      <div class="mt-2">
        כבר יש לך חשבון?
        <router-link to="/login">התחבר כאן</router-link>
      </div>
    </b-form>
  </div>
</template>

<script>
import { reactive, onMounted, ref, computed } from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required, minLength, maxLength, email, sameAs, helpers } from '@vuelidate/validators'
import axios from 'axios'
import countries from '../assets/countries'
import { useRouter } from 'vue-router'

export default {
  name: 'RegisterPage',
  setup() {
    const showPassword = ref(false)
    const showConfirmPassword = ref(false)
    const router = useRouter()

    const state = reactive({
      username: '',
      firstName: '',
      lastName: '',
      email: '',
      country: '',
      password: '',
      confirmPassword: '',
      countries: [],
      submitError: '',
      successMessage: ''
    })

    const passwordValue = computed(() => state.password)

    const rules = {
      username: {
        required: helpers.withMessage('שדה חובה.', required),
        minLength: helpers.withMessage('שם המשתמש חייב להיות לפחות 3 תווים.', minLength(3)),
        maxLength: helpers.withMessage('שם המשתמש לא יכול להיות ארוך מ-8 תווים.', maxLength(8)),
        onlyLetters: helpers.withMessage(
          'שם המשתמש חייב להכיל אותיות באנגלית בלבד.',
          value => /^[A-Za-z]+$/.test(value)
        )
      },
      firstName: { required },
      lastName: { required },
      email: {
        required: helpers.withMessage('שדה חובה.', required),
        email: helpers.withMessage('כתובת אימייל לא תקינה.', email)
      },
      country: { required },
      password: {
        required: helpers.withMessage('שדה חובה.', required),
        minLength: helpers.withMessage('הסיסמה חייבת להיות לפחות באורך 5 תווים.', minLength(5)),
        maxLength: helpers.withMessage('הסיסמה לא יכולה להיות ארוכה מ-10 תווים.', maxLength(10)),
        hasNumber: helpers.withMessage('הסיסמה חייבת לכלול לפחות ספרה אחת.', value => /\d/.test(value)),
        hasSpecialChar: helpers.withMessage('הסיסמה חייבת לכלול לפחות תו מיוחד.', value => /[!@#$%^&*(),.?":{}|<>]/.test(value))
      },
      confirmPassword: {
        required: helpers.withMessage('שדה חובה.', required),
        sameAsPassword: helpers.withMessage('הסיסמאות אינן תואמות.', sameAs(passwordValue))
      }

    }

    const v$ = useVuelidate(rules, state)
    const validations = computed(() => v$.value)

    onMounted(() => {
      state.countries = countries.map(name => ({ value: name, text: name }));
    });

    const handleRegister = async () => {
      state.submitError = ''
      state.successMessage = ''
      v$.value.$touch()
      if (v$.value.$invalid) return

      try {
        const response = await axios.post('http://localhost:3000/Register', {
          username: state.username,
          firstname: state.firstName,
          lastname: state.lastName,
          email: state.email,
          country: state.country,
          password: state.password
        })
        if (response.status === 201) {
          state.successMessage = "ההרשמה בוצעה בהצלחה! מעבירים לעמוד ההתחברות..."
          setTimeout(() => {
            router.push('/login');
          }, 1500);
        } else {
          state.submitError = 'שגיאה לא צפויה בהרשמה'
        }
      } catch (err) {
        const message = (err.response?.data?.message || '').toLowerCase()
        const status = err.response?.status || ''
        if (status === 409 && message.includes('username')) {
          state.submitError = 'שם המשתמש כבר קיים במערכת'
        } else if (status === 409 && message.includes('email')) {
          state.submitError = 'האימייל כבר רשום במערכת'
        } else {
          state.submitError = 'אירעה שגיאה בלתי צפויה. אנא נסה שוב.'
        }
      }
    }

    return {
      state,
      v$,
      validations,
      handleRegister,
      showPassword,
      showConfirmPassword
    }
  }
}
</script>

<style scoped>
.b-form-invalid-feedback {
  font-size: 0.9rem;
}
.b-form-group {
  margin-bottom: 1.25rem;
}
.text-success {
  font-size: 1.1rem;
}
</style>