<script setup lang="ts">
import { useForm } from '@inertiajs/vue3';
import { watch } from 'vue';
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
import { Textarea } from '@/components/ui/textarea';
import { Switch } from '@/components/ui/switch';
import InputError from '@/components/InputError.vue';

type Plan = {
    id: number;
    name: string;
    slug: string;
    description: string;
    price: number;
    currency: string;
    duration_in_days: number;
    is_free: boolean;
    is_active: boolean;
};

type PlanFormModalProps = {
    open: boolean;
    plan?: Plan | null;
};

const props = defineProps<PlanFormModalProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const form = useForm({
    name: '',
    slug: '',
    description: '',
    price: 0,
    currency: 'USD',
    duration_in_days: 30,
    is_free: false,
    is_active: true,
});

// Load plan data when editing
watch(
    () => props.plan,
    (plan) => {
        if (plan) {
            form.name = plan.name;
            form.slug = plan.slug;
            form.description = plan.description;
            form.price = plan.price;
            form.currency = plan.currency;
            form.duration_in_days = plan.duration_in_days;
            form.is_free = plan.is_free;
            form.is_active = plan.is_active;
        } else {
            form.reset();
        }
    },
    { immediate: true }
);

// Auto-generate slug from name
const handleNameChange = (value: string) => {
    form.name = value;
    if (!props.plan) {
        // Only auto-generate slug when creating new plan
        const slug = value
            .toLowerCase()
            .replace(/[^a-z0-9]+/g, '-')
            .replace(/(^-|-$)/g, '');
        form.slug = slug;
    }
};

const handleSubmit = () => {
    if (props.plan) {
        form.put(`/plans/${props.plan.id}`, {
            onSuccess: () => {
                emit('update:open', false);
                form.reset();
            },
        });
    } else {
        form.post('/plans', {
            onSuccess: () => {
                emit('update:open', false);
                form.reset();
            },
        });
    }
};
</script>

<template>
    <Dialog :open="open" @update:open="emit('update:open', $event)">
        <DialogContent class="sm:max-w-[600px]">
            <DialogHeader>
                <DialogTitle>
                    {{ plan ? 'Edit Plan' : 'Create New Plan' }}
                </DialogTitle>
                <DialogDescription>
                    {{
                        plan
                            ? 'Update the plan details below'
                            : 'Fill in the details to create a new plan'
                    }}
                </DialogDescription>
            </DialogHeader>

            <form @submit.prevent="handleSubmit" class="space-y-4">
                <div class="grid gap-4 md:grid-cols-2">
                    <!-- Name -->
                    <div class="space-y-2">
                        <Label for="name">Name *</Label>
                        <Input
                            id="name"
                            :model-value="form.name"
                            @update:model-value="handleNameChange"
                            placeholder="e.g., Professional"
                            required
                        />
                        <InputError :message="form.errors.name" />
                    </div>

                    <!-- Slug -->
                    <div class="space-y-2">
                        <Label for="slug">Slug *</Label>
                        <Input
                            id="slug"
                            v-model="form.slug"
                            placeholder="e.g., professional"
                            required
                        />
                        <InputError :message="form.errors.slug" />
                    </div>
                </div>

                <!-- Description -->
                <div class="space-y-2">
                    <Label for="description">Description</Label>
                    <Textarea
                        id="description"
                        v-model="form.description"
                        placeholder="Brief description of the plan"
                        :rows="3"
                    />
                    <InputError :message="form.errors.description" />
                </div>

                <div class="grid gap-4 md:grid-cols-3">
                    <!-- Price -->
                    <div class="space-y-2">
                        <Label for="price">Price *</Label>
                        <Input
                            id="price"
                            type="number"
                            step="0.01"
                            min="0"
                            v-model.number="form.price"
                            required
                            :disabled="form.is_free"
                        />
                        <InputError :message="form.errors.price" />
                    </div>

                    <!-- Currency -->
                    <div class="space-y-2">
                        <Label for="currency">Currency *</Label>
                        <Input
                            id="currency"
                            v-model="form.currency"
                            placeholder="USD"
                            required
                        />
                        <InputError :message="form.errors.currency" />
                    </div>

                    <!-- Duration -->
                    <div class="space-y-2">
                        <Label for="duration">Duration (days) *</Label>
                        <Input
                            id="duration"
                            type="number"
                            min="1"
                            v-model.number="form.duration_in_days"
                            required
                        />
                        <InputError :message="form.errors.duration_in_days" />
                    </div>
                </div>

                <!-- Switches -->
                <div class="space-y-4">
                    <div class="flex items-center justify-between">
                        <div class="space-y-0.5">
                            <Label for="is_free">Free Plan</Label>
                            <p class="text-sm text-muted-foreground">
                                Mark this plan as free (price will be set to 0)
                            </p>
                        </div>
                        <Switch
                            id="is_free"
                            :checked="form.is_free"
                            @update:checked="
                                (checked) => {
                                    form.is_free = checked;
                                    if (checked) {
                                        form.price = 0;
                                    }
                                }
                            "
                        />
                    </div>

                    <div class="flex items-center justify-between">
                        <div class="space-y-0.5">
                            <Label for="is_active">Active</Label>
                            <p class="text-sm text-muted-foreground">
                                Make this plan available for new subscriptions
                            </p>
                        </div>
                        <Switch
                            id="is_active"
                            :checked="form.is_active"
                            @update:checked="(checked) => (form.is_active = checked)"
                        />
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
                        {{
                            form.processing
                                ? 'Saving...'
                                : plan
                                  ? 'Update Plan'
                                  : 'Create Plan'
                        }}
                    </Button>
                </DialogFooter>
            </form>
        </DialogContent>
    </Dialog>
</template>
