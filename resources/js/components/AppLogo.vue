<script setup lang="ts">
import { usePage } from '@inertiajs/vue3';
import AppLogoIcon from '@/components/AppLogoIcon.vue';
import type { AppPageProps } from '@/types';

const page = usePage<AppPageProps>();
const { name, logo } = page.props;
</script>

<template>
    <div
        class="flex aspect-square size-8 items-center justify-center rounded-md bg-sidebar-primary text-sidebar-primary-foreground overflow-hidden"
    >
        <img
            v-if="logo"
            :src="logo"
            :alt="name"
            class="h-full w-full object-cover"
        />
        <AppLogoIcon v-else class="size-5 fill-current text-white dark:text-black" />
    </div>
    <div class="ml-1 grid flex-1 text-left text-sm">
        <span class="mb-0.5 truncate leading-tight font-semibold">{{ name }}</span>
    </div>
</template>
