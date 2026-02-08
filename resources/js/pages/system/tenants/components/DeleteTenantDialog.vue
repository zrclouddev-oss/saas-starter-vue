<script setup lang="ts">
import { router } from '@inertiajs/vue3';
import { AlertTriangle } from 'lucide-vue-next';
import { ref } from 'vue';
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

type Tenant = {
    id: string;
    name: string;
};

type DeleteTenantDialogProps = {
    open: boolean;
    tenant: Tenant | null;
};

const props = defineProps<DeleteTenantDialogProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const processing = ref(false);
const confirmName = ref('');

const handleDelete = () => {
    if (!props.tenant || confirmName.value !== props.tenant.name) return;

    processing.value = true;
    router.delete(`/tenants/${props.tenant.id}`, {
        preserveScroll: true,
        onSuccess: () => {
            emit('update:open', false);
            confirmName.value = '';
        },
        onFinish: () => (processing.value = false),
    });
};
</script>

<template>
    <Dialog :open="open" @update:open="emit('update:open', $event)">
        <DialogContent>
            <DialogHeader>
                <div class="flex items-center gap-3">
                    <div
                        class="flex h-10 w-10 items-center justify-center rounded-full bg-destructive/10 flex-shrink-0"
                    >
                        <AlertTriangle class="h-5 w-5 text-destructive" />
                    </div>
                    <DialogTitle>Delete Tenant Permanently</DialogTitle>
                </div>
            </DialogHeader>

            <DialogDescription class="space-y-3">
                <p class="text-destructive font-semibold">Warning: This action is irreversible.</p>
                <p>
                    This will permanently delete the tenant
                    <span class="font-semibold text-foreground">{{ tenant?.name }}</span
                    >, including:
                </p>
                <ul class="list-disc pl-4 space-y-1 text-sm bg-muted rounded-md p-3">
                    <li>The tenant database and all its data.</li>
                    <li>All associated user accounts and files.</li>
                    <li>The subdomain configuration.</li>
                </ul>
            </DialogDescription>

            <div class="space-y-2 pt-2">
                <Label for="confirm-name">
                    Type <span class="font-mono font-bold select-all">{{ tenant?.name }}</span> to
                    confirm
                </Label>
                <Input
                    id="confirm-name"
                    v-model="confirmName"
                    placeholder="Type tenant name to confirm"
                />
            </div>

            <DialogFooter>
                <Button
                    type="button"
                    variant="outline"
                    @click="emit('update:open', false)"
                    :disabled="processing"
                >
                    Cancel
                </Button>
                <Button
                    type="button"
                    variant="destructive"
                    @click="handleDelete"
                    :disabled="processing || confirmName !== tenant?.name"
                >
                    {{ processing ? 'Deleting...' : 'Delete Permanently' }}
                </Button>
            </DialogFooter>
        </DialogContent>
    </Dialog>
</template>
