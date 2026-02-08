<script setup lang="ts">
import { Head, router } from '@inertiajs/vue3';
import { ref, watch } from 'vue';
import Heading from '@/components/Heading.vue';
import { Label } from '@/components/ui/label';
import { Switch } from '@/components/ui/switch';
import AppLayout from '@/layouts/AppLayout.vue';
import SettingsLayout from '@/layouts/settings/Layout.vue';
import type { BreadcrumbItem } from '@/types';

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Guest Registration Settings',
        href: '/settings/guest-register',
    },
];

interface Props {
    guest_registration_enabled: boolean;
}

const props = defineProps<Props>();

const enabled = ref(props.guest_registration_enabled);

const toggleGuestRegistration = (checked: boolean) => {
    const previousValue = enabled.value;
    enabled.value = checked;

    router.post(
        '/settings/guest-register',
        { enabled: checked },
        {
            preserveScroll: true,
            preserveState: true,
            onError: () => {
                enabled.value = previousValue; // Revert on error
            },
        },
    );
};
</script>

<template>
    <AppLayout :breadcrumbs="breadcrumbs">
        <Head title="Guest Registration Settings" />

        <SettingsLayout>
            <div class="space-y-6">
                <Heading
                    title="Guest Registration"
                    description="Allow new users to self-register as tenants from the central domain"
                />

                <div class="space-y-4">
                    <div
                        class="flex items-center justify-between gap-4 rounded-lg border p-4"
                    >
                        <div class="flex-1 space-y-0.5">
                            <Label
                                for="guest-registration"
                                class="text-base font-medium"
                            >
                                Enable public registration
                            </Label>
                            <p class="text-sm text-muted-foreground">
                                When enabled, visitors can register at
                                <code
                                    class="rounded bg-muted px-1 text-xs"
                                    >/guest-register</code
                                >
                                and automatically get a subdomain with a free
                                plan.
                            </p>
                        </div>
                        <div class="flex flex-col gap-2">
                            <Switch
                                id="guest-registration"
                                :checked="enabled"
                                @update:checked="toggleGuestRegistration"
                            />
                        </div>
                    </div>

                    <div class="rounded-lg bg-muted p-4 text-sm">
                        <p class="mb-2 font-medium">Registration URLs:</p>
                        <ul class="space-y-1 text-muted-foreground">
                            <li>
                                • <strong>Enabled:</strong>
                                <code class="rounded bg-background px-1"
                                    >/guest-register</code
                                >
                            </li>
                            <li>
                                • <strong>Disabled:</strong>
                                <code class="rounded bg-background px-1"
                                    >/guest-register/disabled</code
                                >
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
        </SettingsLayout>
    </AppLayout>
</template>
