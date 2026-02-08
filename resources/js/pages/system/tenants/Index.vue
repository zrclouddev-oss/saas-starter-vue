<script setup lang="ts">
import { Head, router, usePage } from '@inertiajs/vue3';
import {
    Plus,
    Search,
    Pencil,
    Ban,
    RefreshCw,
    Trash2,
    ExternalLink,
} from 'lucide-vue-next';
import { ref, watch } from 'vue';
import AppLayout from '@/layouts/AppLayout.vue';
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
import { toast } from 'vue-sonner';

// Components
import TenantFormModal from './components/TenantFormModal.vue';
import EditTenantModal from './components/EditTenantModal.vue';
import CancelTenantDialog from './components/CancelTenantDialog.vue';
import RestoreTenantDialog from './components/RestoreTenantDialog.vue';
import DeleteTenantDialog from './components/DeleteTenantDialog.vue';

// Types
type Plan = {
    id: number;
    name: string;
    price_formatted: string;
};

type Tenant = {
    id: string;
    name: string;
    tenancy_db_name: string;
    owner_name: string;
    owner_email: string;
    status: 'Trial' | 'Active' | 'Suspended' | 'Canceled';
    status_badge: 'default' | 'secondary' | 'destructive' | 'outline';
    is_active: boolean;
    plan?: Plan;
    domain?: string;
    domain_url?: string;
    grace_period_days?: number;
    can_restore: boolean;
    can_delete: boolean;
    created_at: string;
};

import type { AppPageProps } from '@/types';

type TenantsPageProps = AppPageProps<{
    tenants: {
        data: Tenant[];
        current_page: number;
        last_page: number;
        per_page: number;
        total: number;
    };
    stats: {
        total: number;
        active: number;
        trial: number;
        canceled: number;
    };
    filters: {
        search?: string;
        status?: string;
    };
    plans: Plan[];
    flash: {
        success?: string;
        error?: string;
    };
}>;

const page = usePage<TenantsPageProps>();
const props = defineProps<TenantsPageProps>();

const search = ref(props.filters.search || '');
const activeTab = ref(props.filters.status?.toLowerCase() || 'all');

// Modal states
const isCreateOpen = ref(false);
const editingTenant = ref<Tenant | null>(null);
const cancelingTenant = ref<Tenant | null>(null);
const restoringTenant = ref<Tenant | null>(null);
const deletingTenant = ref<Tenant | null>(null);

// Flash messages
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
const handleSearch = (value: string) => {
    search.value = value;
    if (value === '') {
        router.get(
            '/tenants',
            { ...props.filters, search: undefined },
            { preserveState: true, replace: true }
        );
    }
};

// Debounce search
let searchTimeout: ReturnType<typeof setTimeout>;
watch(search, (newValue) => {
    if (newValue === props.filters.search) return;

    clearTimeout(searchTimeout);
    searchTimeout = setTimeout(() => {
        router.get(
            '/tenants',
            { ...props.filters, search: newValue || undefined },
            { preserveState: true, replace: true }
        );
    }, 300);
});

// Handle tab change
const handleTabChange = (value: string) => {
    activeTab.value = value;

    const params: any = { search: search.value || undefined };

    if (value !== 'all') {
        // Capitalize first letter for status enum
        params.status = value.charAt(0).toUpperCase() + value.slice(1);
    }

    router.get('/tenants', params, { preserveState: true, replace: true });
};

const breadcrumbs = [
    {
        title: 'Tenants',
        href: '/tenants',
    },
];
</script>

<template>
    <Head title="Tenants" />

    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex h-full flex-1 flex-col gap-4 overflow-x-auto p-4">
            <!-- Header -->
            <div class="flex items-center justify-between">
                <div>
                    <h1 class="text-2xl font-semibold">Tenants</h1>
                    <p class="text-sm text-muted-foreground">
                        Manage your application workspaces
                    </p>
                </div>
                <Button @click="isCreateOpen = true">
                    <Plus class="mr-2 h-4 w-4" />
                    Create Tenant
                </Button>
            </div>

            <!-- Stats Grid -->
            <div class="grid auto-rows-min gap-4 md:grid-cols-4">
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">Total Tenants</p>
                        <p class="text-3xl font-bold">{{ stats.total }}</p>
                    </div>
                </div>
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">Active</p>
                        <p class="text-3xl font-bold text-green-600">{{ stats.active }}</p>
                    </div>
                </div>
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">Trial</p>
                        <p class="text-3xl font-bold text-blue-600">{{ stats.trial }}</p>
                    </div>
                </div>
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6"
                >
                    <div class="space-y-1">
                        <p class="text-sm font-medium text-muted-foreground">Canceled</p>
                        <p class="text-3xl font-bold text-red-600">{{ stats.canceled }}</p>
                    </div>
                </div>
            </div>

            <!-- Filters and Table -->
            <div
                class="relative min-h-[100vh] flex-1 overflow-hidden rounded-xl border border-sidebar-border/70 bg-card md:min-h-min"
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
                                <TabsTrigger value="trial">Trial</TabsTrigger>
                                <TabsTrigger value="suspended">Suspended</TabsTrigger>
                                <TabsTrigger value="canceled">Canceled</TabsTrigger>
                            </TabsList>
                        </Tabs>

                        <div class="relative w-full sm:w-64">
                            <Search
                                class="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground"
                            />
                            <Input
                                placeholder="Search tenants..."
                                v-model="search"
                                @input="handleSearch"
                                class="pl-9"
                            />
                        </div>
                    </div>

                    <!-- Table -->
                    <div class="rounded-md border">
                        <Table>
                            <TableHeader>
                                <TableRow>
                                    <TableHead>Tenant</TableHead>
                                    <TableHead>Owner</TableHead>
                                    <TableHead>Plan</TableHead>
                                    <TableHead>Status</TableHead>
                                    <TableHead>Created</TableHead>
                                    <TableHead class="text-right">Actions</TableHead>
                                </TableRow>
                            </TableHeader>
                            <TableBody>
                                <TableRow v-if="tenants.data.length === 0">
                                    <TableCell colspan="6" class="h-24 text-center">
                                        No tenants found
                                    </TableCell>
                                </TableRow>
                                <TableRow v-for="tenant in tenants.data" :key="tenant.id">
                                    <TableCell>
                                        <div>
                                            <div class="font-medium">{{ tenant.name }}</div>
                                            <a
                                                v-if="tenant.domain_url"
                                                :href="tenant.domain_url"
                                                target="_blank"
                                                rel="noopener noreferrer"
                                                class="text-sm text-blue-600 hover:underline flex items-center gap-1 mt-1"
                                            >
                                                {{ tenant.domain }}
                                                <ExternalLink class="h-3 w-3" />
                                            </a>
                                            <div v-else class="text-sm text-muted-foreground mt-1">
                                                {{ tenant.domain || 'No domain' }}
                                            </div>
                                        </div>
                                    </TableCell>
                                    <TableCell>
                                        <div>
                                            <div class="font-medium">{{ tenant.owner_name }}</div>
                                            <div class="text-sm text-muted-foreground">
                                                {{ tenant.owner_email }}
                                            </div>
                                        </div>
                                    </TableCell>
                                    <TableCell>
                                        <Badge v-if="tenant.plan" variant="outline">
                                            {{ tenant.plan.name }}
                                        </Badge>
                                        <span v-else class="text-muted-foreground text-sm">-</span>
                                    </TableCell>
                                    <TableCell>
                                        <Badge :variant="tenant.status_badge">
                                            {{ tenant.status }}
                                        </Badge>
                                        <div
                                            v-if="
                                                tenant.status === 'Canceled' &&
                                                tenant.grace_period_days != null
                                            "
                                            class="text-xs text-orange-600 mt-1 font-medium"
                                        >
                                            {{ tenant.grace_period_days }} days left
                                        </div>
                                    </TableCell>
                                    <TableCell class="text-sm text-muted-foreground">
                                        {{ new Date(tenant.created_at).toLocaleDateString() }}
                                    </TableCell>
                                    <TableCell class="text-right">
                                        <div class="flex justify-end gap-2">
                                            <Button
                                                variant="ghost"
                                                size="icon"
                                                @click="editingTenant = tenant"
                                                title="Edit"
                                            >
                                                <Pencil class="h-4 w-4" />
                                            </Button>

                                            <Button
                                                v-if="tenant.status !== 'Canceled'"
                                                variant="ghost"
                                                size="icon"
                                                class="text-orange-600 hover:text-orange-700 hover:bg-orange-50"
                                                @click="cancelingTenant = tenant"
                                                title="Cancel Subscription"
                                            >
                                                <Ban class="h-4 w-4" />
                                            </Button>

                                            <Button
                                                v-if="tenant.can_restore"
                                                variant="ghost"
                                                size="icon"
                                                class="text-green-600 hover:text-green-700 hover:bg-green-50"
                                                @click="restoringTenant = tenant"
                                                title="Restore"
                                            >
                                                <RefreshCw class="h-4 w-4" />
                                            </Button>

                                            <Button
                                                v-if="tenant.can_delete"
                                                variant="ghost"
                                                size="icon"
                                                class="text-destructive hover:text-destructive hover:bg-destructive/10"
                                                @click="deletingTenant = tenant"
                                                title="Delete Permanently"
                                            >
                                                <Trash2 class="h-4 w-4" />
                                            </Button>
                                        </div>
                                    </TableCell>
                                </TableRow>
                            </TableBody>
                        </Table>
                    </div>

                    <!-- Pagination -->
                    <div v-if="tenants.last_page > 1" class="flex items-center justify-between">
                        <div class="text-sm text-muted-foreground">
                            Showing {{ (tenants.current_page - 1) * tenants.per_page + 1 }} to
                            {{
                                Math.min(tenants.current_page * tenants.per_page, tenants.total)
                            }}
                            of {{ tenants.total }} results
                        </div>
                        <div class="flex gap-2">
                            <Button
                                variant="outline"
                                size="sm"
                                :disabled="tenants.current_page === 1"
                                @click="
                                    router.get(
                                        `/tenants?page=${tenants.current_page - 1}`,
                                        { ...filters },
                                        { preserveState: true }
                                    )
                                "
                            >
                                Previous
                            </Button>
                            <Button
                                variant="outline"
                                size="sm"
                                :disabled="tenants.current_page === tenants.last_page"
                                @click="
                                    router.get(
                                        `/tenants?page=${tenants.current_page + 1}`,
                                        { ...filters },
                                        { preserveState: true }
                                    )
                                "
                            >
                                Next
                            </Button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Modals -->
        <TenantFormModal
            :open="isCreateOpen"
            @update:open="isCreateOpen = $event"
            :plans="plans"
        />

        <EditTenantModal
            :open="!!editingTenant"
            @update:open="(open) => !open && (editingTenant = null)"
            :tenant="editingTenant"
        />

        <CancelTenantDialog
            :open="!!cancelingTenant"
            @update:open="(open) => !open && (cancelingTenant = null)"
            :tenant="cancelingTenant"
        />

        <RestoreTenantDialog
            :open="!!restoringTenant"
            @update:open="(open) => !open && (restoringTenant = null)"
            :tenant="restoringTenant"
        />

        <DeleteTenantDialog
            :open="!!deletingTenant"
            @update:open="(open) => !open && (deletingTenant = null)"
            :tenant="deletingTenant"
        />
    </AppLayout>
</template>
