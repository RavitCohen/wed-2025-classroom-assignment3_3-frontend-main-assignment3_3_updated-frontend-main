<template>
  <div class="container mt-4" style="max-width: 500px;">
    <h2 class="mb-4">Register</h2>
    <b-form @submit.prevent="handleRegister">
      <b-form-group label="Username" label-for="username">
        <b-form-input
          id="username"
          v-model="state.username"
          @blur="validations.username.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.username.$error">
          <div v-if="!validations.username.required">Username is required.</div>
          <div v-else-if="!validations.username.minLength || !validations.username.maxLength">
            Username must be 3–8 characters.
          </div>
        </b-form-invalid-feedback>
      </b-form-group>

      <b-form-group label="First Name" label-for="firstname">
        <b-form-input id="firstname" v-model="state.firstName" />
      </b-form-group>

      <b-form-group label="Last Name" label-for="lastname">
        <b-form-input id="lastname" v-model="state.lastName" />
      </b-form-group>

      <b-form-group label="Email" label-for="email">
        <b-form-input id="email" type="email" v-model="state.email" />
      </b-form-group>

      <b-form-group label="Country" label-for="country">
        <b-form-select
          id="country"
          v-model="state.country"
          :options="state.countries"
          @change="validations.country.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.country.$error">
          Country is required.
        </b-form-invalid-feedback>
      </b-form-group>

      <b-form-group label="Password" label-for="password">
        <b-form-input
          id="password"
          type="password"
          v-model="state.password"
          @blur="validations.password.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.password.$error">
          <div v-if="!validations.password.required">Password is required.</div>
          <div v-else-if="!validations.password.minLength || !validations.password.maxLength">
            Password must be 5–10 characters.
          </div>
          <div v-else-if="!validations.password.hasNumber">Password must contain at least one number.</div>
          <div v-else-if="!validations.password.hasSpecialChar">Password must contain at least one special character.</div>
        </b-form-invalid-feedback>
      </b-form-group>

      <b-form-group label="Confirm Password" label-for="confirmedPassword">
        <b-form-input
          id="confirmedPassword"
          type="password"
          v-model="state.confirmPassword"
          @blur="validations.confirmPassword.$touch()"
        />
        <b-form-invalid-feedback v-if="validations.confirmPassword.$error">
          <div v-if="!validations.confirmPassword.required">Confirmation is required.</div>
          <div v-else-if="!validations.confirmPassword.sameAsPassword">
            Passwords do not match.
          </div>
        </b-form-invalid-feedback>
      </b-form-group>

      <b-button type="submit" variant="success" class="w-100">Register</b-button>

      <b-alert variant="danger" class="mt-3" dismissible v-if="state.submitError" show>
        Registration failed: {{ state.submitError }}
      </b-alert>

      <div class="mt-2">
        Already have an account?
        <router-link to="/login">Login here</router-link>
      </div>
    </b-form>
  </div>
</template>

<script>
import { reactive, onMounted, nextTick, ref, computed, getCurrentInstance } from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required, minLength, maxLength, email, sameAs } from '@vuelidate/validators'
import countries from '../assets/countries'
import { useRouter } from 'vue-router'

export default {
  name: 'RegisterPage',
  setup() {
    const { proxy } = getCurrentInstance()
    const showPassword = ref(false)
    const showConfirmPassword = ref(false)
    const serverError = ref('')
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
      username: { required, minLength: minLength(3), maxLength: maxLength(8) },
      firstName: { required },
      lastName: { required },
      email: { required, email },
      country: { required },
      password: {
        required,
        minLength: minLength(5),
        maxLength: maxLength(10),
        hasNumber: value => /\d/.test(value) || 'Password must include a number',
        hasSpecialChar: value => /[!@#$%^&*(),.?":{}|<>]/.test(value) || 'Password must include a special character'
      },
      confirmPassword: {
        required,
        sameAsPassword: sameAs(passwordValue, 'Passwords do not match')
      }
    }

    const v$ = useVuelidate(rules, state)
    const validations = computed(() => v$.value)

    onMounted(() => {
      state.countries = countries.map(name => ({ value: name, text: name }));
    });

    const handleRegister = async () => {
      console.log("Start handleRegister function");
      serverError.value = ''
      v$.value.$touch()
      await nextTick()

      if (v$.value.$invalid) {
        console.log('Validation failed on the following fields:');
        for (const [key, field] of Object.entries(v$.value)) {
          if (field?.$invalid) {
            console.warn(`- ${key}:`);
            field.$errors.forEach(err => {
              console.warn(`  Validator "${err.$validator}" failed. Value: ${field.$model}`);
            });
          }
        }
        return;
      }
      try {
        const response = await proxy.axios.post('http://localhost:3000/Register', {
          username: state.username,
          firstname: state.firstName,
          lastname: state.lastName,
          email: state.email,
          country: state.country,
          password: state.password
        })
        console.log("Request to Register sent to Node.js server");
        if (response.status === 201) {
          router.push('/login');
          console.log("Redirect to login page on server side");
        } else {
          serverError.value = 'Unexpected registration error'
        }
      } catch (err) {
        const message = (err.response?.data?.message || '').toLowerCase()
        const status = err.response?.status || ''
        console.error(err);
        if (status === 409 && message.includes('username')) {
          state.submitError = 'Username already exists'
        } else if (status === 409 && message.includes('email')) {
          state.submitError = 'Email already registered'
        } else if (status === 400) {
          state.submitError = err.response?.data?.errors?.[0]?.msg || 'Invalid registration details'
        } else {
          state.submitError = `Unexpected error (${status}). Please try again.`
        }
      }
    }

    return {
      state,
      v$,
      validations,
      handleRegister,
      showPassword,
      showConfirmPassword,
      serverError
    }
  }
}
</script>
