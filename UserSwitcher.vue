<script setup lang="ts">
import type {User} from "../types/user.ts";
import { convertFileSrc } from "@tauri-apps/api/core";

defineProps<{users:User[];currentUserId:number}>();
const emit=defineEmits<{select:[user:User]}>();

function avatarUrl(path: string | null) {
  if (!path) return "";
  if (/^(https?:|data:|asset:|blob:)/.test(path)) return path;
  return convertFileSrc(path);
}

function initials(user: User) {
  return user.display_name?.trim().split(/\s+/).filter(Boolean).slice(0,2).map(part => part[0]?.toUpperCase() ?? "").join("") || "?";
}
</script>
<template>
  <div class="user-switcher">
    <span class="user-switcher__label">Пишет:</span>
    <button v-for="user in users" :key="user.id" type="button" class="user-switcher__button" :class="{'user-switcher__button--active':user.id===currentUserId}" @click="emit('select',user)">
      <span class="user-switcher__avatar">
        <img v-if="user.avatar_path" :src="avatarUrl(user.avatar_path)" alt="" />
        <span v-else>{{ initials(user) }}</span>
      </span>
      <span>{{user.display_name}}</span>
    </button>
  </div>
</template>
<style scoped>
.user-switcher{display:flex;align-items:center;gap:6px}.user-switcher__label{color:var(--muted);font-size:12px}.user-switcher__button{display:inline-flex;align-items:center;gap:7px;padding:6px 10px;border:1px solid var(--border);border-radius:6px;cursor:pointer;background:var(--input);color:var(--text);font:inherit;font-size:12px}.user-switcher__button--active{background:#386be0;border-color:#386be0;color:#fff}.user-switcher__avatar{width:22px;height:22px;border-radius:50%;overflow:hidden;display:inline-flex;align-items:center;justify-content:center;background:var(--hover);font-size:9px;font-weight:700;flex-shrink:0}.user-switcher__avatar img{width:100%;height:100%;object-fit:cover;display:block}
</style>