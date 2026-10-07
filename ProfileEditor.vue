```vue
<script setup lang="ts">
import { ref } from "vue";
import { open } from "@tauri-apps/plugin-dialog";
import { convertFileSrc } from "@tauri-apps/api/core";

import type { ProfileUpdate, User } from "../types/user";
const props = defineProps<{
  user: User;
}>();

const emit = defineEmits<{
  save: [profile: ProfileUpdate];
  close: [];
}>();

const displayName = ref(props.user.display_name);
const username = ref(props.user.username);
const userStatus = ref(props.user.status);
const avatarPath = ref(props.user.avatar_path);

function avatarUrl(path: string | null) {
  if (!path) return "";
  if (/^(https?:|data:|asset:|blob:)/.test(path)) return path;
  return convertFileSrc(path);
}

function getInitials() {
  const name = displayName.value.trim();

  if (!name) {
    return "?";
  }

  return name
      .split(" ")
      .filter(Boolean)
      .slice(0, 2)
      .map((part) => part[0]?.toUpperCase() ?? "")
      .join("");
}

async function chooseAvatar() {
  try {
    const selected = await open({
      multiple: false,
      directory: false,
      filters: [
        {
          name: "Изображения",
          extensions: ["png", "jpg", "jpeg", "webp"],
        },
      ],
    });

    if (typeof selected === "string") {
      avatarPath.value = selected;
    }
  } catch (error) {
    console.error("Ошибка выбора аватара:", error);
  }
}

function removeAvatar() {
  avatarPath.value = null;
}

function submitProfile() {
  const cleanDisplayName = displayName.value.trim();

  const cleanUsername = username.value
      .trim()
      .replace(/^@/, "")
      .replace(/\s/g, "");

  if (!cleanDisplayName || !cleanUsername) {
    return;
  }

  emit("save", {
    displayName: cleanDisplayName,
    username: cleanUsername,
    status: userStatus.value.trim(),
    avatarPath: avatarPath.value,
  });
}
</script>

<template>
  <div
      class="profile-backdrop"
      @click.self="emit('close')"
  >
    <section class="profile-card">

      <!-- Заголовок -->
      <header class="profile-header">
        <div>
          <h2>Профиль</h2>
          <p>Настройте информацию о себе</p>
        </div>

        <button
            type="button"
            class="close-button"
            @click="emit('close')"
        >
          ×
        </button>
      </header>

      <!-- Аватар -->
      <div class="avatar-section">
        <div class="avatar-wrapper">
          <div class="avatar">
            <img
                v-if="avatarPath"
                :src="avatarUrl(avatarPath)"
                alt="Аватар"
            />

            <span v-else>
              {{ getInitials() }}
            </span>
          </div>

          <button
              type="button"
              class="avatar-edit"
              title="Изменить аватар"
              @click="chooseAvatar"
          >
            ✎
          </button>
        </div>

        <div class="avatar-actions">
          <h3>
            {{ displayName || "Новый пользователь" }}
          </h3>

          <span>
            @{{ username || "username" }}
          </span>

          <div class="avatar-buttons">
            <button
                type="button"
                @click="chooseAvatar"
            >
              Изменить аватар
            </button>

            <button
                v-if="avatarPath"
                type="button"
                class="remove-avatar"
                @click="removeAvatar"
            >
              Удалить
            </button>
          </div>
        </div>
      </div>

      <!-- Форма -->
      <form
          class="profile-form"
          @submit.prevent="submitProfile"
      >

        <!-- Имя -->
        <div class="profile-field">
          <label for="profile-display-name">
            Отображаемое имя
          </label>

          <input
              id="profile-display-name"
              v-model="displayName"
              type="text"
              maxlength="40"
              placeholder="Введите имя"
              autocomplete="off"
          />
        </div>

        <!-- Username -->
        <div class="profile-field">
          <label for="profile-username">
            Username
          </label>

          <div class="username-input">
            <span>@</span>

            <input
                id="profile-username"
                v-model="username"
                type="text"
                maxlength="32"
                placeholder="username"
                autocomplete="off"
            />
          </div>

          <span class="field-hint">
            Используйте латинские буквы, цифры и символ _
          </span>
        </div>

        <!-- Статус -->
        <div class="profile-field">
          <label for="profile-status">
            Статус
          </label>

          <textarea
              id="profile-status"
              v-model="userStatus"
              maxlength="120"
              rows="3"
              placeholder="Напишите что-нибудь о себе..."
          ></textarea>

          <span class="character-count">
            {{ userStatus.length }}/120
          </span>
        </div>

        <!-- Информация -->
        <div class="profile-info">
          <div class="info-row">
            <span>Пользователь</span>
            <strong>#{{ user.id }}</strong>
          </div>

          <div class="info-row">
            <span>Аккаунт создан</span>
            <strong>{{ user.created_at }}</strong>
          </div>
        </div>

        <!-- Кнопки -->
        <footer class="profile-actions">
          <button
              type="button"
              class="profile-button secondary"
              @click="emit('close')"
          >
            Отмена
          </button>

          <button
              type="submit"
              class="profile-button primary"
          >
            Сохранить
          </button>
        </footer>

      </form>
    </section>
  </div>
</template>

<style scoped>
.profile-backdrop {
  position: fixed;
  inset: 0;
  z-index: 1000;

  display: flex;
  align-items: center;
  justify-content: center;

  padding: 24px;

  background: rgba(0, 0, 0, 0.68);
  backdrop-filter: blur(8px);
}

.profile-card {
  width: min(500px, 100%);
  max-height: calc(100vh - 48px);

  overflow-y: auto;

  background: var(--panel);
  color: var(--text);

  border: 1px solid var(--border);
  border-radius: 18px;

  box-shadow: 0 25px 70px rgba(0, 0, 0, 0.4);

  animation: profile-open 0.18s ease-out;
}

@keyframes profile-open {
  from {
    opacity: 0;
    transform: translateY(10px) scale(0.98);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* Заголовок */

.profile-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;

  padding: 22px 24px;

  border-bottom: 1px solid var(--border);
}

.profile-header h2 {
  margin: 0;
  font-size: 20px;
}

.profile-header p {
  margin: 5px 0 0;

  color: var(--muted);
  font-size: 13px;
}

.close-button {
  width: 34px;
  height: 34px;

  border: 0;
  border-radius: 9px;

  background: transparent;
  color: var(--muted);

  font-size: 25px;
  line-height: 1;

  cursor: pointer;
}

.close-button:hover {
  background: var(--hover);
  color: var(--text);
}

/* Аватар */

.avatar-section {
  display: flex;
  align-items: center;
  gap: 18px;

  padding: 24px;
}

.avatar-wrapper {
  position: relative;
  flex-shrink: 0;
}

.avatar {
  width: 82px;
  height: 82px;

  display: flex;
  align-items: center;
  justify-content: center;

  overflow: hidden;

  border-radius: 50%;

  background: linear-gradient(
      135deg,
      #5865f2,
      #7c4dff
  );

  color: white;

  font-size: 27px;
  font-weight: 700;
}

.avatar img {
  width: 100%;
  height: 100%;

  object-fit: cover;
}

.avatar-edit {
  position: absolute;
  right: -2px;
  bottom: -2px;

  width: 29px;
  height: 29px;

  display: flex;
  align-items: center;
  justify-content: center;

  border: 3px solid var(--panel);
  border-radius: 50%;

  background: #5865f2;
  color: white;

  cursor: pointer;
}

.avatar-edit:hover {
  background: #4f5be0;
}

.avatar-actions {
  min-width: 0;

  display: flex;
  flex-direction: column;
  gap: 3px;
}

.avatar-actions h3 {
  margin: 0;

  overflow: hidden;

  text-overflow: ellipsis;
  white-space: nowrap;

  font-size: 19px;
}

.avatar-actions > span {
  color: var(--muted);
  font-size: 14px;
}

.avatar-buttons {
  display: flex;
  gap: 7px;

  margin-top: 8px;
}

.avatar-buttons button {
  padding: 6px 9px;

  border: 1px solid var(--border);
  border-radius: 7px;

  background: var(--input);
  color: var(--text);

  font: inherit;
  font-size: 11px;

  cursor: pointer;
}

.avatar-buttons button:hover {
  background: var(--hover);
}

.avatar-buttons .remove-avatar {
  color: #ff6b6b;
}

/* Форма */

.profile-form {
  padding: 0 24px 24px;
}

.profile-field {
  position: relative;

  display: flex;
  flex-direction: column;
  gap: 7px;

  margin-bottom: 18px;
}

.profile-field label {
  font-size: 13px;
  font-weight: 600;
}

.profile-field input,
.profile-field textarea {
  width: 100%;

  border: 1px solid var(--border);
  border-radius: 9px;

  outline: none;

  background: var(--input);
  color: var(--text);

  font: inherit;
  font-size: 14px;
}

.profile-field input {
  height: 42px;
  padding: 0 13px;
}

.profile-field textarea {
  min-height: 82px;
  padding: 11px 13px;

  resize: vertical;
}

.profile-field input:focus,
.profile-field textarea:focus {
  border-color: #5865f2;

  box-shadow:
      0 0 0 3px rgba(88, 101, 242, 0.12);
}

.profile-field input::placeholder,
.profile-field textarea::placeholder {
  color: var(--muted);
}

/* Username */

.username-input {
  display: flex;
  align-items: center;

  height: 42px;

  border: 1px solid var(--border);
  border-radius: 9px;

  background: var(--input);

  overflow: hidden;
}

.username-input:focus-within {
  border-color: #5865f2;

  box-shadow:
      0 0 0 3px rgba(88, 101, 242, 0.12);
}

.username-input > span {
  padding-left: 13px;

  color: var(--muted);
}

.username-input input {
  height: 40px;

  border: 0;
  border-radius: 0;

  background: transparent;

  box-shadow: none;
}

.username-input input:focus {
  border: 0;
  box-shadow: none;
}

.field-hint {
  color: var(--muted);
  font-size: 11px;
}

.character-count {
  position: absolute;
  right: 10px;
  bottom: 9px;

  color: var(--muted);
  font-size: 10px;
}

/* Информация */

.profile-info {
  margin: 5px 0 20px;

  padding: 5px 14px;

  border: 1px solid var(--border);
  border-radius: 10px;

  background: var(--input);
}

.info-row {
  display: flex;
  justify-content: space-between;
  align-items: center;

  padding: 10px 0;

  font-size: 12px;
}

.info-row + .info-row {
  border-top: 1px solid var(--border);
}

.info-row span {
  color: var(--muted);
}

.info-row strong {
  color: var(--text);
  font-weight: 600;
}

/* Кнопки */

.profile-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.profile-button {
  min-width: 105px;
  height: 40px;

  padding: 0 16px;

  border-radius: 9px;

  font: inherit;
  font-size: 13px;
  font-weight: 600;

  cursor: pointer;
}

.profile-button.secondary {
  border: 1px solid var(--border);

  background: var(--input);
  color: var(--text);
}

.profile-button.secondary:hover {
  background: var(--hover);
}

.profile-button.primary {
  border: 1px solid #5865f2;

  background: #5865f2;
  color: white;
}

.profile-button.primary:hover {
  background: #4f5be0;
}

/* Мобильная версия */

@media (max-width: 600px) {
  .profile-backdrop {
    padding: 12px;
  }

  .profile-card {
    max-height: calc(100vh - 24px);
  }

  .avatar-section,
  .profile-header {
    padding-left: 18px;
    padding-right: 18px;
  }

  .profile-form {
    padding-left: 18px;
    padding-right: 18px;
  }

  .profile-actions {
    flex-direction: column-reverse;
  }

  .profile-button {
    width: 100%;
  }
}
</style>

