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
import InputError from '@/components/InputError.vue';
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from '@/components/ui/select';

type Tenant = {
    id: string;
    name: string;
    owner_name: string;
    owner_email: string;
    status: string;
};

type EditTenantModalProps = {
    open: boolean;
    tenant: Tenant | null;
};

const props = defineProps<EditTenantModalProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const form = useForm({
    owner_name: '',
    owner_email: '',
    status: '',
});

watch(
    () => props.tenant,
    (tenant) => {
        if (tenant) {
            form.owner_name = tenant.owner_name;
            form.owner_email = tenant.owner_email;
            form.status = tenant.status;
        }
    },
    { immediate: true }
);

const handleSubmit = () => {
    if (!props.tenant) return;

    form.put(`/tenants/${props.tenant.id}`, {
        onSuccess: () => {
            emit('update:open', false);
            form.reset();
        },
    });
};
</script>

<template>
    <Dialog :open="open" @update:open="emit('update:open', $event)">
        <DialogContent class="sm:max-w-[500px]">
            <DialogHeader>
                <DialogTitle>Edit Tenant: {{ tenant?.name }}</DialogTitle>
                <DialogDescription> Update tenant basic information and status. </DialogDescription>
            </DialogHeader>

            <form @submit.prevent="handleSubmit" class="space-y-4">
                <div class="space-y-2">
                    <Label for="owner_name">Owner Name</Label>
                    <Input id="owner_name" v-model="form.owner_name" required />
                    <InputError :message="form.errors.owner_name" />
                </div>

                <div class="space-y-2">
                    <Label for="owner_email">Owner Email</Label>
                    <Input id="owner_email" type="email" v-model="form.owner_email" required />
                    <InputError :message="form.errors.owner_email" />
                </div>

                <div class="space-y-2">
                    <Label for="status">Status</Label>
                    <Select v-model="form.status">
                        <SelectTrigger>
                            <SelectValue placeholder="Select status" />
                        </SelectTrigger>
                        <SelectContent>
                            <SelectItem value="Trial">Trial</SelectItem>
                            <SelectItem value="Active">Active</SelectItem>
                            <SelectItem value="Suspended">Suspended</SelectItem>
                        </SelectContent>
                    </Select>
                    <p class="text-xs text-muted-foreground">
                        Note: Canceled status is handled via the separate Cancel action.
                    </p>
                    <InputError :message="form.errors.status" />
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
                    <Button type="submit" :disabled="form.processing"> Save Changes </Button>
                </DialogFooter>
            </form>
        </DialogContent>
    </Dialog>
</template>
