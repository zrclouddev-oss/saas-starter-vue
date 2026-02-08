<script setup lang="ts">
import { router } from '@inertiajs/vue3';
import { RefreshCw } from 'lucide-vue-next';
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

type Tenant = {
    id: string;
    name: string;
};

type RestoreTenantDialogProps = {
    open: boolean;
    tenant: Tenant | null;
};

const props = defineProps<RestoreTenantDialogProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const processing = ref(false);

const handleRestore = () => {
    if (!props.tenant) return;

    processing.value = true;
    router.post(
        `/tenants/${props.tenant.id}/restore`,
        {},
        {
            preserveScroll: true,
            onSuccess: () => emit('update:open', false),
            onFinish: () => (processing.value = false),
        }
    );
};
</script>

<template>
    <Dialog :open="open" @update:open="emit('update:open', $event)">
        <DialogContent>
            <DialogHeader>
                <div class="flex items-center gap-3">
                    <div
                        class="flex h-10 w-10 items-center justify-center rounded-full bg-green-100 flex-shrink-0"
                    >
                        <RefreshCw class="h-5 w-5 text-green-600" />
                    </div>
                    <DialogTitle>Restore Tenant</DialogTitle>
                </div>
                <DialogDescription>
                    This will check if the grace period is still valid and restore access for
                    <span class="font-semibold">{{ tenant?.name }}</span
                    >.
                </DialogDescription>
            </DialogHeader>

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
                    class="bg-green-600 hover:bg-green-700"
                    @click="handleRestore"
                    :disabled="processing"
                >
                    {{ processing ? 'Restoring...' : 'Confirm Restore' }}
                </Button>
            </DialogFooter>
        </DialogContent>
    </Dialog>
</template>
