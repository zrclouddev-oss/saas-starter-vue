<script setup lang="ts">
import { Head, useForm, router } from '@inertiajs/vue3';
import { ref } from 'vue';
import Heading from '@/components/Heading.vue';
import InputError from '@/components/InputError.vue';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import AppLayout from '@/layouts/AppLayout.vue';
import SettingsLayout from '@/layouts/settings/Layout.vue';
import type { BreadcrumbItem } from '@/types';

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'General Settings',
        href: '/settings/general',
    },
];

interface Props {
    app_name: string;
    app_logo: string | null;
    settings_general_update_url: string;
}

const props = defineProps<Props>();

const form = useForm({
    app_name: props.app_name,
    app_logo: null as File | null,
});

const fileInput = ref<HTMLInputElement | null>(null);
const logoPreview = ref<string | null>(null);

const selectNewLogo = () => {
    fileInput.value?.click();
};

const updateLogoPreview = (event: Event) => {
    const target = event.target as HTMLInputElement;
    const photo = target.files?.[0];

    if (!photo) return;

    form.app_logo = photo;
    const reader = new FileReader();
    reader.onload = (e) => {
        logoPreview.value = e.target?.result as string;
    };
    reader.readAsDataURL(photo);
};

const submit = () => {
    form.post(props.settings_general_update_url, {
        preserveScroll: true,
        onSuccess: () => {
            if (fileInput.value) {
                fileInput.value.value = '';
            }
            form.app_logo = null;
            logoPreview.value = null;
            
            // Reload shared props to update logo and name in sidebar without full page refresh
            router.reload({ only: ['name', 'logo'] });
        },
    });
};
</script>

<template>
    <AppLayout :breadcrumbs="breadcrumbs">
        <Head title="General Settings" />

        <SettingsLayout>
            <div class="space-y-6">
                <Heading
                    title="General Settings"
                    description="Update your application's public identity."
                />

                <form @submit.prevent="submit" class="space-y-6">
                    <div class="space-y-2">
                        <Label for="app_name">Application Name</Label>
                        <Input
                            id="app_name"
                            v-model="form.app_name"
                            type="text"
                            class="mt-1 block w-full"
                            required
                            autofocus
                            autocomplete="organization"
                        />
                        <InputError class="mt-2" :message="form.errors.app_name" />
                    </div>

                    <div class="space-y-2">
                        <Label for="app_logo">Application Logo</Label>

                        <!-- Current Logo Preview -->
                        <div class="mt-2">
                            <div
                                v-if="logoPreview"
                                class="mb-4 h-20 w-20 rounded-md border bg-cover bg-center bg-no-repeat"
                                :style="{ backgroundImage: `url('${logoPreview}')` }"
                            />
                            <div v-else-if="app_logo" class="mb-4">
                                <img
                                    :src="`/storage/${app_logo}`"
                                    alt="Current Logo"
                                    class="h-20 w-auto rounded-md border object-contain p-2"
                                />
                            </div>
                            <div
                                v-else
                                class="mb-4 flex h-20 w-20 items-center justify-center rounded-md border bg-muted text-muted-foreground"
                            >
                                No Logo
                            </div>
                        </div>

                        <input
                            ref="fileInput"
                            type="file"
                            class="hidden"
                            accept="image/*"
                            @change="updateLogoPreview"
                        />

                        <Button
                            type="button"
                            variant="secondary"
                            @click="selectNewLogo"
                        >
                            Select A New Logo
                        </Button>

                        <InputError class="mt-2" :message="form.errors.app_logo" />
                    </div>

                    <div class="flex items-center gap-4">
                        <Button type="submit" :disabled="form.processing">Save</Button>

                        <Transition
                            enter-active-class="transition ease-in-out"
                            enter-from-class="opacity-0"
                            leave-active-class="transition ease-in-out"
                            leave-to-class="opacity-0"
                        >
                            <p
                                v-if="form.recentlySuccessful"
                                class="text-sm text-gray-600 dark:text-gray-400"
                            >
                                Saved.
                            </p>
                        </Transition>
                    </div>
                </form>
            </div>
        </SettingsLayout>
    </AppLayout>
</template>
