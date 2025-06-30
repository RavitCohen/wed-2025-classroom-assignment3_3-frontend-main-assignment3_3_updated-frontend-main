<template>
  <div class="container mt-4" style="max-width: 500px;">
    <h2 class="mb-4">הרשמה</h2>
    <b-form @submit.prevent="handleRegister">
      <!-- שם משתמש -->
      <b-form-group label="שם משתמש" label-for="username">
        <b-form-input
          id="username"
          v-model="state.username"
          :state="validations.username.$dirty ? !validations.username.$invalid : null"
          @blur="validations.username.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.username.$error">
          <div v-if="!validations.username.required">שדה חובה.</div>
          <div v-else-if="!validations.username.minLength || !validations.username.maxLength">
            שם המשתמש חייב להיות בין 3–8 תווים.
          </div>
          <div v-else-if="!validations.username.onlyLetters">
           שם המשתמש חייב להכיל אותיות בלבד.
          </div>

        </b-form-invalid-feedback>
      </b-form-group>

      <!-- שם פרטי -->
      <b-form-group label="שם פרטי" label-for="firstname">
        <b-form-input
          id="firstname"
          v-model="state.firstName"
          :state="validations.firstName.$dirty ? !validations.firstName.$invalid : null"
          @blur="validations.firstName.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.firstName.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- שם משפחה -->
      <b-form-group label="שם משפחה" label-for="lastname">
        <b-form-input
          id="lastname"
          v-model="state.lastName"
          :state="validations.lastName.$dirty ? !validations.lastName.$invalid : null"
          @blur="validations.lastName.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.lastName.$error">
          שדה חובה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- אימייל -->
      <b-form-group label="אימייל" label-for="email">
        <b-form-input
          id="email"
          type="email"
          v-model="state.email"
          :state="validations.email.$dirty ? !validations.email.$invalid : null"
          @blur="validations.email.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.email.$error">
          כתובת אימייל לא תקינה.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- מדינה -->
      <b-form-group label="מדינה" label-for="country">
        <b-form-select
          id="country"
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
      <b-form-group label="סיסמה" label-for="password">
        <b-form-input
          id="password"
          :type="showPassword ? 'text' : 'password'"
          v-model="state.password"
          :state="validations.password.$dirty ? !validations.password.$invalid : null"
          @blur="validations.password.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.password.$error">
          <div v-if="!validations.password.required">שדה חובה.</div>
          <div v-else-if="!validations.password.minLength || !validations.password.maxLength">
            הסיסמה חייבת להיות בין 5–10 תווים.
          </div>
          <div v-else-if="!validations.password.hasNumber">הסיסמה חייבת לכלול לפחות ספרה אחת.</div>
          <div v-else-if="!validations.password.hasSpecialChar">הסיסמה חייבת לכלול לפחות תו מיוחד.</div>
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- אימות סיסמה -->
      <b-form-group label="אימות סיסמה" label-for="confirmedPassword">
        <b-form-input
          id="confirmedPassword"
          :type="showConfirmPassword ? 'text' : 'password'"
          v-model="state.confirmPassword"
          :state="validations.confirmPassword.$dirty ? !validations.confirmPassword.$invalid : null"
          @blur="validations.confirmPassword.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.confirmPassword.$error">
          <div v-if="!validations.confirmPassword.required">שדה חובה.</div>
          <div v-else-if="!validations.confirmPassword.sameAsPassword">הסיסמאות אינן תואמות.</div>
        </b-form-invalid-feedback>
      </b-form-group>

      <b-button type="submit" variant="success" class="w-100">הרשמה</b-button>

      <b-alert variant="danger" class="mt-3" dismissible v-if="state.submitError" show>
        {{ state.submitError }}
      </b-alert>

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
import { required, minLength, maxLength, email, sameAs } from '@vuelidate/validators'
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
      submitError: ''
    })

    const passwordValue = computed(() => state.password)

    const rules = {
      username: {
        required,
        minLength: minLength(3),
        maxLength: maxLength(8),
        onlyLetters: value => /^[A-Za-z]+$/.test(value)
      },
      firstName: { required },
      lastName: { required },
      email: { required, email },
      country: { required },
      password: {
        required,
        minLength: minLength(5),
        maxLength: maxLength(10),
        hasNumber: value => /\d/.test(value),
        hasSpecialChar: value => /[!@#$%^&*(),.?":{}|<>]/.test(value)
      },
      confirmPassword: {
        required,
        sameAsPassword: sameAs(passwordValue)
      }
    }

    const v$ = useVuelidate(rules, state)
    const validations = computed(() => v$.value)

    onMounted(() => {
      state.countries = countries.map(name => ({ value: name, text: name }));
    });

    const handleRegister = async () => {
      state.submitError = ''
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
          router.push('/login');
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
