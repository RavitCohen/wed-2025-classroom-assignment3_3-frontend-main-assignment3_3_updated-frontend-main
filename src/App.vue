<template>
  <div id="app" dir="rtl">
    <b-navbar toggleable="lg" type="dark" variant="info" class="px-4" dir="rtl">
      <!-- צד ימין: לוגו + ניווט -->
      <b-navbar-brand :to="{ name: 'main' }" class="font-weight-bold text-white">
        מתכוני סבתא
      </b-navbar-brand>

      <b-navbar-toggle target="nav-collapse" />

      <b-collapse id="nav-collapse" is-nav class="w-100 d-flex justify-content-between">
        <!-- קישורים מימין -->
        <b-navbar-nav class="d-flex align-items-center">
          <router-link class="nav-link" :to="{ name: 'main' }">דף הבית</router-link>
          <router-link class="nav-link" :to="{ name: 'search' }">חיפוש</router-link>
          <router-link class="nav-link" :to="{ name: 'about' }">אודות</router-link>
        </b-navbar-nav>

        <!-- אזור משתמש משמאל -->
        <b-navbar-nav class="d-flex align-items-center">
          <template v-if="!store.username">
            <b-navbar-text class="text-white mx-2">שלום אורח</b-navbar-text>
            <router-link class="nav-link" :to="{ name: 'login' }">התחברות</router-link>
            <router-link class="nav-link" :to="{ name: 'register' }">הרשמה</router-link>
          </template>

          <template v-else>
            <b-navbar-text class="text-white mx-2">{{ store.username }}</b-navbar-text>
            <b-nav-item-dropdown text="אזור אישי" right>
              <b-dropdown-item :to="{ name: 'favorites' }">המועדפים שלי</b-dropdown-item>
              <b-dropdown-item :to="{ name: 'myRecipes' }">המתכונים שלי</b-dropdown-item>
              <b-dropdown-item :to="{ name: 'familyRecipes' }">המשפחתיים שלי</b-dropdown-item>
            </b-nav-item-dropdown>
            <b-nav-item @click="showCreateModal = true">מתכון חדש</b-nav-item>
            <b-nav-item @click="logout">התנתקות</b-nav-item>
          </template>
        </b-navbar-nav>
      </b-collapse>
    </b-navbar>

    <CreateRecipeModal v-model:show="showCreateModal" />

    <router-view />
  </div>
</template>

<script>
import { getCurrentInstance, ref } from 'vue';
import CreateRecipeModal from "@/components/CreateRecipeModal.vue";

export default {
  name: "App",
  components: {
    CreateRecipeModal
  },
  setup() {
    const internalInstance = getCurrentInstance();
    const store = internalInstance.appContext.config.globalProperties.store;
    const toast = internalInstance.appContext.config.globalProperties.toast;
    const router = internalInstance.appContext.config.globalProperties.$router;

    const showCreateModal = ref(false);

    const logout = () => {
      store.logout();
      toast("Logout", "User logged out successfully", "success");
      router.push("/").catch(() => {});
    };

    return { store, logout, showCreateModal };
  }
};
</script>

<style lang="scss">
@import "@/scss/form-style.scss";

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  min-height: 100vh;
}

.nav-link {
  color: white !important;
  font-weight: 500;
  margin: 0 10px;
  transition: 0.2s;

  &:hover {
    color: #dff6ff !important;
    text-decoration: underline;
  }
}

.b-navbar {
  font-family: 'Heebo', sans-serif;
  font-size: 1rem;
}
</style>
