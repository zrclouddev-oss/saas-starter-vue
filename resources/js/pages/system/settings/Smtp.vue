<script setup lang="ts">
import { Head, useForm, router } from '@inertiajs/vue3';
import { Mail, Send } from 'lucide-vue-next';
import Heading from '@/components/Heading.vue';
import InputError from '@/components/InputError.vue';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from '@/components/ui/select';
import AppLayout from '@/layouts/AppLayout.vue';
import SettingsLayout from '@/layouts/settings/Layout.vue';
import type { BreadcrumbItem } from '@/types';
import systemSettings from '@/routes/system/settings';

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'SMTP Settings',
        href: '/settings/smtp',
    },
];

interface Props {
    smtp_host: string;
    smtp_port: string;
    smtp_encryption: string;
    smtp_username: string;
    smtp_from_address: string;
    smtp_from_name: string;
}

const props = defineProps<Props>();

const form = useForm({
    smtp_host: props.smtp_host,
    smtp_port: props.smtp_port,
    smtp_encryption: props.smtp_encryption,
    smtp_username: props.smtp_username,
    smtp_password: '',
    smtp_from_address: props.smtp_from_address,
    smtp_from_name: props.smtp_from_name,
});

const submit = () => {
    form.post(systemSettings.smtp.url(), {
        preserveScroll: true,
        onSuccess: () => {
            form.smtp_password = ''; // Clear password field after save
        },
    });
};

const sendTestEmail = () => {
    router.post(systemSettings.smtp.test.url(), {}, {
        preserveScroll: true,
        preserveState: true,
    });
};
</script>

<template>
    <AppLayout :breadcrumbs="breadcrumbs">
        <Head title="SMTP Settings" />

        <SettingsLayout>
            <div class="space-y-6">
                <Heading
                    title="SMTP Configuration"
                    description="Configure your email server settings for sending system emails."
                />

                <form @submit.prevent="submit" class="space-y-6">
                    <div class="grid grid-cols-1 gap-6 md:grid-cols-2">
                        <!-- SMTP Host -->
                        <div class="space-y-2">
                            <Label for="smtp_host">SMTP Host</Label>
                            <Input
                                id="smtp_host"
                                v-model="form.smtp_host"
                                type="text"
                                placeholder="smtp.gmail.com"
                                required
                            />
                            <InputError :message="form.errors.smtp_host" />
                        </div>

                        <!-- SMTP Port -->
                        <div class="space-y-2">
                            <Label for="smtp_port">Port</Label>
                            <Input
                                id="smtp_port"
                                v-model="form.smtp_port"
                                type="number"
                                placeholder="587"
                                required
                            />
                            <InputError :message="form.errors.smtp_port" />
                        </div>

                        <!-- Encryption -->
                        <div class="space-y-2">
                            <Label for="smtp_encryption">Encryption</Label>
                            <Select v-model="form.smtp_encryption">
                                <SelectTrigger>
                                    <SelectValue placeholder="Select encryption" />
                                </SelectTrigger>
                                <SelectContent>
                                    <SelectItem value="tls">TLS</SelectItem>
                                    <SelectItem value="ssl">SSL</SelectItem>
                                    <SelectItem value="none">None</SelectItem>
                                </SelectContent>
                            </Select>
                            <InputError :message="form.errors.smtp_encryption" />
                        </div>

                        <!-- Username -->
                        <div class="space-y-2">
                            <Label for="smtp_username">Username</Label>
                            <Input
                                id="smtp_username"
                                v-model="form.smtp_username"
                                type="text"
                                placeholder="your-email@example.com"
                                required
                            />
                            <InputError :message="form.errors.smtp_username" />
                        </div>

                        <!-- Password -->
                        <div class="space-y-2">
                            <Label for="smtp_password">Password</Label>
                            <Input
                                id="smtp_password"
                                v-model="form.smtp_password"
                                type="password"
                                placeholder="Leave empty to keep current"
                            />
                            <p class="text-xs text-muted-foreground">
                                Leave empty to keep the current password
                            </p>
                            <InputError :message="form.errors.smtp_password" />
                        </div>

                        <!-- From Address -->
                        <div class="space-y-2">
                            <Label for="smtp_from_address">From Address</Label>
                            <Input
                                id="smtp_from_address"
                                v-model="form.smtp_from_address"
                                type="email"
                                placeholder="noreply@example.com"
                                required
                            />
                            <InputError :message="form.errors.smtp_from_address" />
                        </div>

                        <!-- From Name -->
                        <div class="space-y-2 md:col-span-2">
                            <Label for="smtp_from_name">From Name</Label>
                            <Input
                                id="smtp_from_name"
                                v-model="form.smtp_from_name"
                                type="text"
                                placeholder="My Application"
                                required
                            />
                            <InputError :message="form.errors.smtp_from_name" />
                        </div>
                    </div>

                    <div class="flex items-center gap-4">
                        <Button type="submit" :disabled="form.processing">
                            <Mail class="mr-2 h-4 w-4" />
                            Save Settings
                        </Button>

                        <Button
                            type="button"
                            variant="outline"
                            :disabled="form.processing"
                            @click="sendTestEmail"
                        >
                            <Send class="mr-2 h-4 w-4" />
                            Send Test Email
                        </Button>

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
