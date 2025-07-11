<template>
  <div id="app" dir="rtl">
    <b-navbar toggleable="lg" type="dark" variant="dark" class="main-navbar">
      <!-- Brand -->
      <b-navbar-brand :to="{ name: 'main' }" class="brand">
        🍲 מתכוני סבתא
      </b-navbar-brand>

        <b-navbar-toggle target="nav-collapse" />
        
        <b-collapse id="nav-collapse" is-nav class="w-100">

          <b-navbar-nav class="align-items-center w-100">
          <!-- קישורים ראשיים -->
          <router-link class="nav-link" :to="{ name: 'main' }">
            <i class="bi bi-house"></i> דף הבית
          </router-link>
          <router-link class="nav-link" :to="{ name: 'search' }">
            <i class="bi bi-search"></i> חיפוש
          </router-link>
          <router-link class="nav-link" :to="{ name: 'about' }">
            <i class="bi bi-info-circle"></i> אודות
          </router-link>

          <!-- Spacer -->
          <div class="flex-fill"></div>

          <!-- אזור משתמש -->
          <template v-if="!isLoggedIn">
            <b-navbar-text class="text-light mx-2">שלום אורח</b-navbar-text>
            <router-link class="nav-link" :to="{ name: 'login' }">
              <i class="bi bi-box-arrow-in-right"></i> התחברות
            </router-link>
            <router-link class="nav-link" :to="{ name: 'register' }">
              <i class="bi bi-person-plus"></i> הרשמה
            </router-link>
          </template>

          <template v-else>
          <b-navbar-text class="user-greeting mx-2">
            שלום {{ store.username }} ! &nbsp;
          </b-navbar-text>
            <div
              class="position-relative"
              @click.stop="toggleDropdown"
              style="cursor: pointer; color: white;"
              aria-label="אזור אישי"
            >
              אזור אישי ▼
              <transition name="fade-slide">
                <div
                  v-show="isShowDropdown"
                  class="custom-dropdown"
                  @click.stop
                >
                <router-link class="dropdown-link" :to="{ name: 'favorites' }" @click="closeDropdown">
                  ❤️ המועדפים שלי
                </router-link>
                <router-link class="dropdown-link" :to="{ name: 'myRecipes' }" @click="closeDropdown">
                  📝 המתכונים שלי
                </router-link>
                <router-link class="dropdown-link" :to="{ name: 'familyRecipes' }" @click="closeDropdown">
                  👨‍👩‍👧‍👦 המשפחתיים שלי
                </router-link>

                </div>
              </transition>
            </div>

            <b-nav-item @click="showCreateModal = true">
              ➕ מתכון חדש
            </b-nav-item>
            <b-nav-item @click="logout">
              🚪 התנתקות
            </b-nav-item>
          </template>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>

    <CreateRecipeModal v-model:show="showCreateModal" />

    <router-view />
  </div>
</template>

<script>
import { getCurrentInstance, ref, computed, onMounted, onUnmounted } from 'vue';
import CreateRecipeModal from "@/components/CreateRecipeModal.vue";
import store from '@/store';
import axios from "axios";

export default {
  name: "App",
  components: { CreateRecipeModal },
  setup() {
    const internalInstance = getCurrentInstance();
    const router = internalInstance.appContext.config.globalProperties.$router;
    const isLoggedIn = computed(() => !!store.username.value);
    const showCreateModal = ref(false);
    const isShowDropdown = ref(false);

    const toggleDropdown = () => {
      isShowDropdown.value = !isShowDropdown.value;
    };

    const closeDropdown = () => {
      isShowDropdown.value = false;
    };

    const logout = async () => {
      store.logout();
      await axios.post('http://localhost:3000/Logout', {}, { withCredentials: true });
      router.push({ name: 'login', query: { logout: '1' } }).catch(() => {});
    };


    onMounted(() => {
      document.addEventListener('click', closeDropdown);
    });
    onUnmounted(() => {
      document.removeEventListener('click', closeDropdown);
    });

    return {
      store,
      logout,
      showCreateModal,
      isLoggedIn,
      isShowDropdown,
      toggleDropdown,
      closeDropdown,
    };
  }
};
</script>

<style lang="scss">
@import "@/scss/form-style.scss";

#app {
  font-family: 'Heebo', sans-serif;
  min-height: 100vh;
  color: #2c3e50;
}

/* Navbar */
.main-navbar {
  background: linear-gradient(90deg, #0d6efd 0%, #6610f2 100%);
  padding: 0.5rem 1.5rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.brand {
  font-size: 1.3rem;
  font-weight: bold;
  color: #ffffff !important;
}

.nav-link {
  color: #ffffff !important;
  margin: 0 0.7rem;
  font-weight: 500;
  position: relative;
  transition: color 0.2s;
}

.nav-link::after {
  content: '';
  position: absolute;
  bottom: -4px;
  left: 0;
  width: 0%;
  height: 2px;
  background-color: #ffffff;
  transition: width 0.3s ease;
}

.nav-link:hover::after {
  width: 100%;
}

.b-navbar-nav .nav-link {
  display: flex;
  align-items: center;
}

.b-navbar-nav .bi {
  margin-left: 0.3rem;
}

.custom-dropdown {
  position: absolute;
  top: calc(100% + 0.5rem);
  background-color: #f0f8ff;
  border: 1px solid #bcd;
  border-radius: 0.8rem;
  min-width: 200px;
  padding: 0.5rem 0;
  box-shadow: 0 8px 20px rgba(0,0,0,0.1);
  z-index: 1000;
  text-align: center; /* מרכז הטקסט */
}


.dropdown-link {
  display: block;
  padding: 0.25rem 0;
  color: #007bff;
  text-decoration: none;

  &:hover {
    text-decoration: underline;
    background-color: #f8f9fa;
  }
}

.custom-dropdown::before {
  content: "";
  position: absolute;
  top: -8px;
  right: 1.5rem;
  border-width: 0 8px 8px 8px;
  border-style: solid;
  border-color: transparent transparent #f0f8ff transparent;
  filter: drop-shadow(0 -1px 1px rgba(0,0,0,0.1));
}


@media (max-width: 992px) {
  .main-navbar .nav-link {
    display: block;
    text-align: right;
    padding: 0.75rem 1rem;
    margin: 0;
    border-bottom: 1px solid rgba(121, 93, 207, 0.15);
  }

}

body {
  font-family: 'Heebo', sans-serif;
  font-weight: 400;
  line-height: 1.8;
  color: #333;
}

body {
  background: linear-gradient(rgba(255, 255, 255, 0.7), rgba(255, 255, 255, 0.7)),
             url('~@/assets/background.jpg') no-repeat center center fixed;
  background-size: cover;
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.25s ease;
}
.fade-slide-enter-from,
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-5px);
}
.fade-slide-enter-to,
.fade-slide-leave-from {
  opacity: 1;
  transform: translateY(0);
}
.user-greeting {
  color: #fff;
  background-color: rgba(255, 255, 255, 0.1);
  padding: 4px 10px;
  border-radius: 10px;
  font-weight: 500;
  display: inline-block;
  box-shadow: 0 1px 3px rgba(0,0,0,0.4);
}




</style>