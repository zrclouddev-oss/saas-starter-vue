<script setup lang="ts">
import { useForm } from '@inertiajs/vue3';
import { watch } from 'vue';
import InputError from '@/components/InputError.vue';
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
import tenant from '@/routes/tenant';

interface User {
    id: number;
    name: string;
    email: string;
}

interface Props {
    show: boolean;
    user?: User | null;
}

const props = defineProps<Props>();
const emit = defineEmits<{
    close: [];
}>();

const form = useForm({
    name: '',
    email: '',
    password: '',
    password_confirmation: '',
});

// Watch for user changes to populate form
watch(
    () => props.user,
    (newUser) => {
        if (newUser) {
            form.name = newUser.name;
            form.email = newUser.email;
            form.password = '';
            form.password_confirmation = '';
        } else {
            form.reset();
        }
    },
    { immediate: true }
);

// Watch for show changes to reset form
watch(
    () => props.show,
    (newShow) => {
        if (!newShow) {
            form.reset();
            form.clearErrors();
        }
    }
);

const submit = () => {
    if (props.user) {
        // Update existing user
        form.put(`/users/${props.user.id}`, {
            preserveScroll: true,
            onSuccess: () => {
                emit('close');
            },
        });
    } else {
        // Create new user
        form.post('/users', {
            preserveScroll: true,
            onSuccess: () => {
                emit('close');
            },
        });
    }
};

const handleClose = () => {
    if (!form.processing) {
        emit('close');
    }
};
</script>

<template>
    <Dialog :open="show" @update:open="handleClose">
        <DialogContent class="sm:max-w-[500px]">
            <DialogHeader>
                <DialogTitle>{{ user ? 'Edit User' : 'Create User' }}</DialogTitle>
                <DialogDescription>
                    {{ user ? 'Update user information.' : 'Add a new user to this tenant.' }}
                </DialogDescription>
            </DialogHeader>

            <form @submit.prevent="submit" class="space-y-4">
                <!-- Name -->
                <div class="space-y-2">
                    <Label for="name">Name</Label>
                    <Input
                        id="name"
                        v-model="form.name"
                        type="text"
                        placeholder="John Doe"
                        required
                    />
                    <InputError :message="form.errors.name" />
                </div>

                <!-- Email -->
                <div class="space-y-2">
                    <Label for="email">Email</Label>
                    <Input
                        id="email"
                        v-model="form.email"
                        type="email"
                        placeholder="john@example.com"
                        required
                    />
                    <InputError :message="form.errors.email" />
                </div>

                <!-- Password -->
                <div class="space-y-2">
                    <Label for="password">Password</Label>
                    <Input
                        id="password"
                        v-model="form.password"
                        type="password"
                        :placeholder="user ? 'Leave empty to keep current' : 'Enter password'"
                        :required="!user"
                    />
                    <p v-if="user" class="text-xs text-muted-foreground">
                        Leave empty to keep the current password
                    </p>
                    <InputError :message="form.errors.password" />
                </div>

                <!-- Password Confirmation -->
                <div class="space-y-2">
                    <Label for="password_confirmation">Confirm Password</Label>
                    <Input
                        id="password_confirmation"
                        v-model="form.password_confirmation"
                        type="password"
                        placeholder="Confirm password"
                        :required="!user || form.password.length > 0"
                    />
                    <InputError :message="form.errors.password_confirmation" />
                </div>

                <DialogFooter>
                    <Button
                        type="button"
                        variant="outline"
                        @click="handleClose"
                        :disabled="form.processing"
                    >
                        Cancel
                    </Button>
                    <Button type="submit" :disabled="form.processing">
                        {{ user ? 'Update User' : 'Create User' }}
                    </Button>
                </DialogFooter>
            </form>
        </DialogContent>
    </Dialog>
</template>
