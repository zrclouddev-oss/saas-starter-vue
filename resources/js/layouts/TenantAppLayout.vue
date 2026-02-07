<script setup lang="ts">
import TenantAppLayout from '@/layouts/app/TenantAppSidebarLayout.vue';
import type { BreadcrumbItem } from '@/types';

type Props = {
    breadcrumbs?: BreadcrumbItem[];
};

withDefaults(defineProps<Props>(), {
    breadcrumbs: () => [],
});
</script>

<template>
    <TenantAppLayout :breadcrumbs="breadcrumbs">
        <slot />
    </TenantAppLayout>
</template>
