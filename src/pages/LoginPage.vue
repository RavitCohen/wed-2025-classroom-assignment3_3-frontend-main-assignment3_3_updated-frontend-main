<template>
  <div class="container mt-5 login-container" style="max-width: 400px;" dir="rtl">
    <h2 class="mb-4 text-center">התחברות</h2>
    <b-form @submit.prevent="login">
      <!-- Username -->
      <b-form-group label="שם משתמש" label-for="username">
        <b-form-input
          id="username"
          v-model="state.username"
          :state="getValidationState(v$.username)"
          placeholder="👤 הכנס שם משתמש"
        />
        <b-form-invalid-feedback v-if="v$.username.$error">
          חובה להזין שם משתמש.
        </b-form-invalid-feedback>
      </b-form-group>

      <!-- Password -->
      <b-form-group label="סיסמה" label-for="password">
        <b-form-input
          id="password"
          type="password"
          v-model="state.password"
          :state="getValidationState(v$.password)"
          placeholder="🔒 הכנס סיסמה"
        />
        <b-form-invalid-feedback v-if="v$.password.$error">
          חובה להזין סיסמה.
        </b-form-invalid-feedback>
      </b-form-group>

      <b-button type="submit" variant="primary" class="w-100">התחבר</b-button>

      <!-- Alert לשם משתמש/סיסמה שגויים -->
      <b-alert
        variant="info"
        class="mt-3"
        dismissible
        v-if="state.invalidCredentials"
        show
      >
        שם המשתמש או הסיסמה אינם נכונים. בדוק ונסה שוב.
      </b-alert>

      <!-- Alert לשגיאות כלליות -->
      <b-alert
        variant="danger"
        class="mt-3"
        dismissible
        v-if="state.submitError"
        show
      >
        ההתחברות נכשלה: {{ state.submitError }}
      </b-alert>

      <div class="mt-3 text-center">
        אין לך חשבון?
        <router-link to="/register" class="register-link">
          להרשמה לחץ כאן
        </router-link>
      </div>
    </b-form>
  </div>
</template>

<script>
import { reactive } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required } from '@vuelidate/validators';
import axios from 'axios';
import { useRouter } from 'vue-router';
import store from '../store';

export default {
  name: 'LoginPage',
  setup() {
    const router = useRouter();
    const state = reactive({
      username: '',
      password: '',
      submitError: null,
      invalidCredentials: false,
    });

    const rules = {
      username: { required },
      password: { required },
    };

    const v$ = useVuelidate(rules, state);

    const getValidationState = (field) => {
      return field.$dirty ? !field.$invalid : null;
    };

    const login = async () => {
      state.invalidCredentials = false;
      state.submitError = null;

      const valid = await v$.value.$validate();
      if (!valid) return;

      try {
        await axios.post('http://localhost:3000/Login', {
          username: state.username,
          password: state.password,
        });
        store.login(state.username);
        router.push('/');
      } catch (err) {
        const status = err.response?.status;
        const message = err.response?.data?.message;

        if (
          status === 401 ||
          message === 'Invalid credentials' ||
          message === 'Unauthorized'
        ) {
          state.invalidCredentials = true;
          alert("שם המשתמש או הסיסמה לא נכונים.");
        } else if (message) {
          state.submitError = message;
        } else {
          state.submitError = 'שגיאה בלתי צפויה. נסה שוב.';
        }
      }
    };

    return { state, v$, login, getValidationState };
  },
};
</script>

<style scoped>
.login-container {
  background-color: #fff;
  padding: 2rem;
  border-radius: 0.75rem;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  font-family: 'Heebo', sans-serif;
}
.register-link {
  color: #0d6efd;
  font-weight: 500;
  text-decoration: none;
}
.register-link:hover {
  text-decoration: underline;
}
</style>
