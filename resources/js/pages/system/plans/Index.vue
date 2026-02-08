<script setup lang="ts">
import { Head, router, usePage } from '@inertiajs/vue3';
import { Plus, Search } from 'lucide-vue-next';
import { ref, watch } from 'vue';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
} from '@/components/ui/table';
import { Tabs, TabsList, TabsTrigger } from '@/components/ui/tabs';
import AppLayout from '@/layouts/AppLayout.vue';
import type { BreadcrumbItem, AppPageProps } from '@/types';
import { toast } from 'vue-sonner';
import PlanFormModal from './components/PlanFormModal.vue';
import DeletePlanDialog from './components/DeletePlanDialog.vue';

type Plan = {
    id: number;
    name: string;
    slug: string;
    description: string;
    price: number;
    price_formatted: string;
    currency: string;
    duration_in_days: number;
    duration_text: string;
    is_free: boolean;
    is_active: boolean;
    tenants_count: number;
};

type PlansPageProps = AppPageProps<{
    plans: {
        data: Plan[];
        current_page: number;
        last_page: number;
        per_page: number;
        total: number;
    };
    filters: {
        search?: string;
        is_active?: string;
        is_free?: string;
    };
    stats: {
        total: number;
        active: number;
        inactive: number;
    };
    flash?: {
        success?: string;
        error?: string;
    };
}>;

const page = usePage<PlansPageProps>();
const props = defineProps<PlansPageProps>();

const search = ref(props.filters.search || '');
const activeTab = ref('all');
const isCreateModalOpen = ref(false);
const editingPlan = ref<Plan | null>(null);
const deletingPlan = ref<Plan | null>(null);

// Show flash messages
watch(
    () => page.props.flash,
    (flash) => {
        if (flash?.success) {
            toast.success(flash.success as string);
        }
        if (flash?.error) {
            toast.error(flash.error as string);
        }
    },
    { deep: true }
);

// Handle search
const handleSearch = (value: string | number) => {
    const stringValue = String(value);
    search.value = stringValue;
    router.get(
        '/plans',
        { search: stringValue || undefined },
        { preserveState: true, replace: true }
    );
};

// Handle tab change
const handleTabChange = (value: string | number) => {
    const stringValue = String(value);
    activeTab.value = stringValue;
    const params: Record<string, string> = { search: search.value };

    if (stringValue === 'active') {
        params.is_active = '1';
    } else if (stringValue === 'inactive') {
        params.is_active = '0';
    }

    router.get('/plans', params, {
        preserveState: true,
        replace: true,
    });
};

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Plans',
        href: '/plans',
    },
];
</script>

<template>
    <Head title="Plans" />

    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex h-full flex-1 flex-col gap-4 overflow-x-auto p-4">
            <!-- Header -->
            <div class="flex items-center justify-between">
                <div>
                    <h1 class="text-2xl font-semibold">Plans</h1>
                    <p class="text-sm text-muted-foreground">
                        Manage your subscription plans
                    </p>
                </div>
                <Button @click="isCreateModalOpen = true">
                    <Plus class="mr-2 h-4 w-4" />
                    Create Plan
                </Button>
            </div>

            <!-- Stats -->
            <div class="grid auto-rows-min gap-4 md:grid-cols-3">
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 dark:border-sidebar-border"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">
                            Total Plans
                        </p>
                        <p class="text-3xl font-bold">{{ stats.total }}</p>
                    </div>
                </div>
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 dark:border-sidebar-border"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">
                            Active Plans
                        </p>
                        <p class="text-3xl font-bold text-green-600 dark:text-green-500">
                            {{ stats.active }}
                        </p>
                    </div>
                </div>
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 dark:border-sidebar-border"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">
                            Inactive Plans
                        </p>
                        <p class="text-3xl font-bold text-red-600 dark:text-red-500">
                            {{ stats.inactive }}
                        </p>
                    </div>
                </div>
            </div>

            <!-- Filters and Table -->
            <div
                class="relative min-h-[100vh] flex-1 overflow-hidden rounded-xl border border-sidebar-border/70 bg-card md:min-h-min dark:border-sidebar-border"
            >
                <div class="p-6 space-y-4">
                    <!-- Filters -->
                    <div
                        class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between"
                    >
                        <Tabs :model-value="activeTab" @update:model-value="handleTabChange">
                            <TabsList>
                                <TabsTrigger value="all">All</TabsTrigger>
                                <TabsTrigger value="active">Active</TabsTrigger>
                                <TabsTrigger value="inactive">Inactive</TabsTrigger>
                            </TabsList>
                        </Tabs>

                        <div class="relative w-full sm:w-64">
                            <Search
                                class="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground"
                            />
                            <Input
                                placeholder="Search plans..."
                                :model-value="search"
                                @update:model-value="handleSearch"
                                class="pl-9"
                            />
                        </div>
                    </div>

                    <!-- Table -->
                    <div class="rounded-md border">
                        <Table>
                            <TableHeader>
                                <TableRow>
                                    <TableHead>Name</TableHead>
                                    <TableHead>Price</TableHead>
                                    <TableHead>Duration</TableHead>
                                    <TableHead>Status</TableHead>
                                    <TableHead>Tenants</TableHead>
                                    <TableHead class="text-right">Actions</TableHead>
                                </TableRow>
                            </TableHeader>
                            <TableBody>
                                <TableRow v-if="plans.data.length === 0">
                                    <TableCell colspan="6" class="h-24 text-center">
                                        No plans found
                                    </TableCell>
                                </TableRow>
                                <TableRow v-for="plan in plans.data" :key="plan.id">
                                    <TableCell>
                                        <div class="font-medium">{{ plan.name }}</div>
                                        <div class="text-sm text-muted-foreground">
                                            {{ plan.description }}
                                        </div>
                                    </TableCell>
                                    <TableCell>
                                        <div class="font-medium">
                                            {{ plan.price_formatted }}
                                        </div>
                                        <Badge v-if="plan.is_free" variant="secondary" class="mt-1">
                                            Free
                                        </Badge>
                                    </TableCell>
                                    <TableCell>{{ plan.duration_text }}</TableCell>
                                    <TableCell>
                                        <Badge :variant="plan.is_active ? 'default' : 'secondary'">
                                            {{ plan.is_active ? 'Active' : 'Inactive' }}
                                        </Badge>
                                    </TableCell>
                                    <TableCell>{{ plan.tenants_count }}</TableCell>
                                    <TableCell class="text-right">
                                        <div class="flex justify-end gap-2">
                                            <Button
                                                variant="outline"
                                                size="sm"
                                                @click="editingPlan = plan"
                                            >
                                                Edit
                                            </Button>
                                            <Button
                                                variant="outline"
                                                size="sm"
                                                @click="deletingPlan = plan"
                                            >
                                                Delete
                                            </Button>
                                        </div>
                                    </TableCell>
                                </TableRow>
                            </TableBody>
                        </Table>
                    </div>

                    <!-- Pagination -->
                    <div v-if="plans.last_page > 1" class="flex items-center justify-between">
                        <div class="text-sm text-muted-foreground">
                            Showing {{ (plans.current_page - 1) * plans.per_page + 1 }} to
                            {{ Math.min(plans.current_page * plans.per_page, plans.total) }} of
                            {{ plans.total }} results
                        </div>
                        <div class="flex gap-2">
                            <Button
                                variant="outline"
                                size="sm"
                                :disabled="plans.current_page === 1"
                                @click="router.get(`/plans?page=${plans.current_page - 1}`)"
                            >
                                Previous
                            </Button>
                            <Button
                                variant="outline"
                                size="sm"
                                :disabled="plans.current_page === plans.last_page"
                                @click="router.get(`/plans?page=${plans.current_page + 1}`)"
                            >
                                Next
                            </Button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Modals -->
        <PlanFormModal
            :open="isCreateModalOpen || !!editingPlan"
            @update:open="
                (open) => {
                    if (!open) {
                        isCreateModalOpen = false;
                        editingPlan = null;
                    }
                }
            "
            :plan="editingPlan"
        />
        <DeletePlanDialog :plan="deletingPlan" @update:open="() => (deletingPlan = null)" />
    </AppLayout>
</template>
