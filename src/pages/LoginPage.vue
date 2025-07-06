<template>
  <div class="container mt-5 login-container" style="max-width: 400px;" dir="rtl">
    <h2 class="mb-4 text-center">התחברות</h2>
    <b-form @submit.prevent="login">
      <!-- Username -->
      <b-form-group label="שם משתמש" label-for="username">
        <b-form-input
          id="username"
          v-model="state.username"
          @input="hideLogoutMessage"
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
          @input="hideLogoutMessage"
          placeholder="🔒 הכנס סיסמה"
        />
        <b-form-invalid-feedback v-if="v$.password.$error">
          חובה להזין סיסמה.
        </b-form-invalid-feedback>
      </b-form-group>

      <b-button type="submit" variant="primary" class="w-100">התחבר</b-button>

      <!-- הודעת התנתקות -->
      <div
        v-if="logoutSuccess"
        class="alert alert-info mt-3 text-center"
        role="alert"
      >
        התנתקת בהצלחה.
      </div>

      <!-- הודעת שגיאה על סיסמה שגויה -->
      <div
        v-if="state.invalidCredentials"
        class="alert alert-danger mt-3 text-center"
        role="alert"
      >
        שם המשתמש או הסיסמה שגויים. נסה שוב.
      </div>

      <!-- הודעת הצלחה -->
      <div
        v-if="state.loginSuccess"
        class="alert alert-success mt-3 text-center"
        role="alert"
      >
        התחברת בהצלחה!
      </div>

      <!-- הודעת שגיאה כללית -->
      <div
        v-if="state.submitError"
        class="alert alert-danger mt-3 text-center"
        role="alert"
      >
        ההתחברות נכשלה: {{ state.submitError }}
      </div>
    </b-form>

    <div class="mt-3 text-center">
      אין לך חשבון?
      <router-link to="/register" class="register-link">
        להרשמה לחץ כאן
      </router-link>
    </div>
  </div>
</template>

<script>
import { reactive, computed } from 'vue';
import { useVuelidate } from '@vuelidate/core';
import { required } from '@vuelidate/validators';
import axios from 'axios';
import { useRouter, useRoute } from 'vue-router';
import store from '../store';

export default {
  name: 'LoginPage',
  setup() {
    const router = useRouter();
    const route = useRoute();

    const state = reactive({
      username: '',
      password: '',
      submitError: null,
      invalidCredentials: false,
      loginSuccess: false,
      hideLogoutSuccess: false
    });

    const logoutSuccess = computed(() => route.query.logout === '1' && !state.hideLogoutSuccess);

    const rules = {
      username: { required },
      password: { required },
    };

    const v$ = useVuelidate(rules, state);

    const getValidationState = (field) => {
      return field.$dirty ? !field.$invalid : null;
    };

    const hideLogoutMessage = () => {
      state.hideLogoutSuccess = true;
    };

    const login = async () => {
      state.invalidCredentials = false;
      state.submitError = null;
      state.loginSuccess = false;

      const valid = await v$.value.$validate();
      if (!valid) return;

      try {
        await axios.post('http://localhost:3000/Login', {
          username: state.username,
          password: state.password,
        });

        state.loginSuccess = true;

        setTimeout(() => {
          store.login(state.username);
          router.push('/');
        }, 1500);
      } catch (err) {
        const status = err.response?.status;
        const message = err.response?.data?.message;

        if (status === 401) {
          state.invalidCredentials = true;
        } else if (message) {
          state.submitError = message;
        } else {
          state.submitError = 'שגיאה בלתי צפויה. נסה שוב.';
        }
      }
    };

    return {
      state,
      v$,
      login,
      getValidationState,
      logoutSuccess,
      hideLogoutMessage
    };
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
