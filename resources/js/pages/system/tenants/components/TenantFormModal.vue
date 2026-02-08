<script setup lang="ts">
import { useForm } from '@inertiajs/vue3';
import { computed } from 'vue';
import { Button } from '@/components/ui/button';
import {
    Dialog,
    DialogContent,
    DialogDescription,
    DialogFooter,
    DialogHeader,
    DialogTitle,
} from '@/components/ui/dialog';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import InputError from '@/components/InputError.vue';
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from '@/components/ui/select';

type Plan = {
    id: number;
    name: string;
    price_formatted: string;
};

type TenantFormModalProps = {
    open: boolean;
    plans: Plan[];
};

const props = defineProps<TenantFormModalProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const form = useForm({
    name: '',
    domain: '',
    owner_name: '',
    owner_email: '',
    owner_password: '',
    owner_password_confirmation: '',
    plan_id: '',
    status: 'Active', // Default status
});

const hostname = computed(() => window.location.hostname);

const handleSubmit = () => {
    form.post('/tenants', {
        onSuccess: () => {
            emit('update:open', false);
            form.reset();
        },
    });
};
</script>


<template>
    <Dialog :open="open" @update:open="emit('update:open', $event)">
        <DialogContent class="sm:max-w-[600px]">
            <DialogHeader>
                <DialogTitle>Create New Tenant</DialogTitle>
                <DialogDescription>
                    Create a new tenant workspace with a dedicated database and domain.
                </DialogDescription>
            </DialogHeader>

            <form @submit.prevent="handleSubmit" class="space-y-4">
                <div class="grid gap-4 md:grid-cols-2">
                    <!-- Tenant Name -->
                    <div class="space-y-2">
                        <Label for="name">Tenant Name *</Label>
                        <Input
                            id="name"
                            v-model="form.name"
                            placeholder="e.g., Acme Corp"
                            required
                        />
                        <InputError :message="form.errors.name" />
                    </div>

                    <!-- Domain -->
                    <div class="space-y-2">
                        <Label for="domain">Domain (Subdomain) *</Label>
                        <div class="flex">
                            <Input
                                id="domain"
                                v-model="form.domain"
                                placeholder="acme"
                                class="rounded-r-none"
                                required
                            />
                            <div
                                class="flex items-center rounded-r-md border border-l-0 bg-muted px-3 text-sm text-muted-foreground"
                            >
                                .{{ hostname }}
                            </div>
                        </div>
                        <InputError :message="form.errors.domain" />
                    </div>
                </div>

                <div class="space-y-4 border-t pt-4">
                    <h4 class="font-medium">Owner Information</h4>
                    <div class="grid gap-4 md:grid-cols-2">
                        <div class="space-y-2">
                            <Label for="owner_name">Owner Name *</Label>
                            <Input
                                id="owner_name"
                                v-model="form.owner_name"
                                placeholder="John Doe"
                                required
                            />
                            <InputError :message="form.errors.owner_name" />
                        </div>
                        <div class="space-y-2">
                            <Label for="owner_email">Owner Email *</Label>
                            <Input
                                id="owner_email"
                                type="email"
                                v-model="form.owner_email"
                                placeholder="john@example.com"
                                required
                            />
                            <InputError :message="form.errors.owner_email" />
                        </div>
                    </div>

                    <div class="grid gap-4 md:grid-cols-2">
                        <div class="space-y-2">
                            <Label for="owner_password">Password *</Label>
                            <Input
                                id="owner_password"
                                type="password"
                                v-model="form.owner_password"
                                required
                            />
                            <InputError :message="form.errors.owner_password" />
                        </div>
                        <div class="space-y-2">
                            <Label for="owner_password_confirmation">Confirm Password *</Label>
                            <Input
                                id="owner_password_confirmation"
                                type="password"
                                v-model="form.owner_password_confirmation"
                                required
                            />
                        </div>
                    </div>
                </div>

                <div class="grid gap-4 md:grid-cols-2 border-t pt-4">
                    <div class="space-y-2">
                        <Label for="plan_id">Plan *</Label>
                        <Select v-model="form.plan_id" required>
                            <SelectTrigger>
                                <SelectValue placeholder="Select a plan" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem
                                    v-for="plan in plans"
                                    :key="plan.id"
                                    :value="plan.id.toString()"
                                >
                                    {{ plan.name }} ({{ plan.price_formatted }})
                                </SelectItem>
                            </SelectContent>
                        </Select>
                        <InputError :message="form.errors.plan_id" />
                    </div>

                    <div class="space-y-2">
                        <Label for="status">Initial Status</Label>
                        <Select v-model="form.status">
                            <SelectTrigger>
                                <SelectValue placeholder="Select status" />
                            </SelectTrigger>
                            <SelectContent>
                                <SelectItem value="Trial">Trial</SelectItem>
                                <SelectItem value="Active">Active</SelectItem>
                            </SelectContent>
                        </Select>
                        <InputError :message="form.errors.status" />
                    </div>
                </div>

                <DialogFooter>
                    <Button
                        type="button"
                        variant="outline"
                        @click="emit('update:open', false)"
                        :disabled="form.processing"
                    >
                        Cancel
                    </Button>
                    <Button type="submit" :disabled="form.processing">
                        {{ form.processing ? 'Creating...' : 'Create Tenant' }}
                    </Button>
                </DialogFooter>
            </form>
        </DialogContent>
    </Dialog>
</template>
