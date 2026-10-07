<script setup lang="ts">
import { onMounted, ref } from "vue";
import UserSwitcher from "./UserSwitcher.vue";
import type { User } from "../types/user";

defineProps<{
  status: string;
  users: User[];
  currentUser: User;
}>();

const emit = defineEmits<{
  select: [user: User];
  profile: [];
}>();

const isDark = ref(true);

onMounted(() => {
  isDark.value =
      localStorage.getItem("encore-theme") !== "light";

  applyTheme();
});

function applyTheme() {
  document.documentElement.dataset.theme =
      isDark.value ? "dark" : "light";

  localStorage.setItem(
      "encore-theme",
      isDark.value ? "dark" : "light"
  );
}

function toggleTheme() {
  isDark.value = !isDark.value;
  applyTheme();
}
</script>

<template>
  <header class="header">
    <div>
      <h1>Encore 67 messenger</h1>
      <p>{{ status }}</p>
    </div>

    <div class="header__actions">
      <UserSwitcher
          :users="users"
          :current-user-id="currentUser.id"
          @select="emit('select', $event)"
      />

      <button
          type="button"
          class="theme-button"
          @click="toggleTheme"
      >
        {{ isDark ? "☀️" : "🌙" }}
        {{ isDark ? "Светлая" : "Тёмная" }}
      </button>

      <button
          type="button"
          class="profile-open-button"
          @click="emit('profile')"
      >
        Профиль
      </button>

      <span class="badge">Локально</span>
    </div>
  </header>
</template>

<style scoped>
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 24px;
  border-bottom: 1px solid var(--border);
  background: var(--panel);
  color: var(--text);
  flex-shrink: 0;
}

.header__actions {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.header h1 {
  margin: 0;
  font-size: 18px;
}

.header p {
  margin: 4px 0 0;
  color: var(--muted);
}

.badge,
.theme-button,
.profile-open-button {
  padding: 7px 10px;
  border: 1px solid var(--border);
  border-radius: 7px;
  background: var(--input);
  color: var(--text);
  font: inherit;
  font-size: 12px;
  cursor: pointer;
}

.theme-button:hover,
.profile-open-button:hover {
  background: var(--hover);
}
</style>