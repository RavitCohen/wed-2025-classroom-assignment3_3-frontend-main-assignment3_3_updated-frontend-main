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
            <b-navbar-text class="text-light mx-2 font-weight-bold">{{ store.username }}</b-navbar-text>
            <div
              class="position-relative"
              @click.stop="toggleDropdown"
              style="cursor: pointer; color: white;"
              aria-label="אזור אישי"
            >
              אזור אישי ▼
              <div
                v-show="isShowDropdown"
                class="custom-dropdown"
                @click.stop
              >
                <router-link class="dropdown-link" :to="{ name: 'favorites' }">
                  ❤️ המועדפים שלי
                </router-link>
                <router-link class="dropdown-link" :to="{ name: 'myRecipes' }">
                  📝 המתכונים שלי
                </router-link>
                <router-link class="dropdown-link" :to="{ name: 'familyRecipes' }">
                  👨‍👩‍👧‍👦 המשפחתיים שלי
                </router-link>
              </div>
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
      alert("התנתקת בהצלחה");
      router.push
      router.push("/login").catch(() => {});
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
      closeDropdown
    };
  }
};
</script>

<style lang="scss">
@import "@/scss/form-style.scss";

#app {
  font-family: 'Heebo', sans-serif;
  min-height: 100vh;
  background: #f6f8fa;
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

/* Dropdown Custom */
.custom-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  background-color: white;
  border: 1px solid #ccc;
  z-index: 1000;
  min-width: 150px;
  padding: 0.5rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
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

@media (max-width: 992px) {
  .nav-link {
    margin: 0.5rem 0;
  }
}
</style>
