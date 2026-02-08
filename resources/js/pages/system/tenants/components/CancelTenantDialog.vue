<script setup lang="ts">
import { router } from '@inertiajs/vue3';
import { AlertCircle } from 'lucide-vue-next';
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

type CancelTenantDialogProps = {
    open: boolean;
    tenant: Tenant | null;
};

const props = defineProps<CancelTenantDialogProps>();
const emit = defineEmits<{
    (e: 'update:open', value: boolean): void;
}>();

const processing = ref(false);

const handleCancel = () => {
    if (!props.tenant) return;

    processing.value = true;
    router.post(
        `/tenants/${props.tenant.id}/cancel`,
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
                        class="flex h-10 w-10 items-center justify-center rounded-full bg-orange-100 flex-shrink-0"
                    >
                        <AlertCircle class="h-5 w-5 text-orange-600" />
                    </div>
                    <DialogTitle>Cancel Tenant Subscription</DialogTitle>
                </div>
            </DialogHeader>

            <DialogDescription class="space-y-3 pt-2">
                <p>
                    Are you sure you want to cancel the subscription for
                    <span class="font-semibold text-foreground">{{ tenant?.name }}</span
                    >?
                </p>
                <div class="rounded-md bg-muted p-3 text-sm">
                    <ul class="list-disc pl-4 space-y-1">
                        <li>The tenant will be deactivated immediately.</li>
                        <li>
                            A <strong>30-day grace period</strong> will start.
                        </li>
                        <li>The data will be permanently deleted after the grace period ends.</li>
                        <li>You can restore the tenant anytime during the grace period.</li>
                    </ul>
                </div>
            </DialogDescription>

            <DialogFooter>
                <Button
                    type="button"
                    variant="outline"
                    @click="emit('update:open', false)"
                    :disabled="processing"
                >
                    Keep Active
                </Button>
                <Button
                    type="button"
                    variant="destructive"
                    @click="handleCancel"
                    :disabled="processing"
                >
                    {{ processing ? 'Canceling...' : 'Cancel Subscription' }}
                </Button>
            </DialogFooter>
        </DialogContent>
    </Dialog>
</template>
