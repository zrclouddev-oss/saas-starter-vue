<script setup lang="ts">
import { Head } from '@inertiajs/vue3';
import AppearanceTabs from '@/components/AppearanceTabs.vue';
import Heading from '@/components/Heading.vue';
import TenantAppLayout from '@/layouts/TenantAppLayout.vue';
import TenantLayout from '@/layouts/settings/TenantLayout.vue';
import tenant from '@/routes/tenant';
import { type BreadcrumbItem } from '@/types';

const breadcrumbItems: BreadcrumbItem[] = [
    {
        title: 'Appearance settings',
        href: tenant.settings.appearance.edit().url,
    },
];
</script>

<template>
    <TenantAppLayout :breadcrumbs="breadcrumbItems">
        <Head title="Appearance settings" />

        <h1 class="sr-only">Appearance Settings</h1>

        <TenantLayout>
            <div class="space-y-6">
                <Heading
                    variant="small"
                    title="Appearance settings"
                    description="Update your account's appearance settings"
                />
                <AppearanceTabs />
            </div>
        </TenantLayout>
    </TenantAppLayout>
</template>
