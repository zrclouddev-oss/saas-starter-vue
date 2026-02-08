<script setup lang="ts">
import { router } from '@inertiajs/vue3';
import { AlertTriangle } from 'lucide-vue-next';
import { ref } from 'vue';
import {
    Dialog,
    DialogContent,
    DialogDescription,
    DialogFooter,
    DialogHeader,
    DialogTitle,
} from '@/components/ui/dialog';
import { Button } from '@/components/ui/button';

type Plan = {
    id: number;
    name: string;
    tenants_count: number;
};

type DeletePlanDialogProps = {
    plan: Plan | null;
};

const props = defineProps<DeletePlanDialogProps>();
const emit = defineEmits<{
    (e: 'update:open'): void;
}>();

const isDeleting = ref(false);

const handleDelete = () => {
    if (!props.plan) return;

    isDeleting.value = true;
    router.delete(`/plans/${props.plan.id}`, {
        onSuccess: () => {
            emit('update:open');
        },
        onFinish: () => {
            isDeleting.value = false;
        },
    });
};
</script>

<template>
    <Dialog :open="!!plan" @update:open="emit('update:open')">
        <DialogContent>
            <DialogHeader>
                <div class="flex items-center gap-3">
                    <div
                        class="flex h-10 w-10 items-center justify-center rounded-full bg-destructive/10"
                    >
                        <AlertTriangle class="h-5 w-5 text-destructive" />
                    </div>
                    <DialogTitle>Delete Plan</DialogTitle>
                </div>
                <DialogDescription>
                    Are you sure you want to delete the plan
                    <span class="font-semibold">{{ plan?.name }}</span
                    >?
                </DialogDescription>
            </DialogHeader>

            <div
                v-if="plan && plan.tenants_count > 0"
                class="rounded-lg border border-destructive/50 bg-destructive/10 p-4"
            >
                <p class="text-sm text-destructive">
                    <strong>Warning:</strong> This plan has
                    {{ plan.tenants_count }} active tenant(s). You cannot delete a plan with
                    active tenants.
                </p>
            </div>

            <DialogFooter>
                <Button
                    type="button"
                    variant="outline"
                    @click="emit('update:open')"
                    :disabled="isDeleting"
                >
                    Cancel
                </Button>
                <Button
                    type="button"
                    variant="destructive"
                    @click="handleDelete"
                    :disabled="isDeleting || (plan?.tenants_count ?? 0) > 0"
                >
                    {{ isDeleting ? 'Deleting...' : 'Delete Plan' }}
                </Button>
            </DialogFooter>
        </DialogContent>
    </Dialog>
</template>
