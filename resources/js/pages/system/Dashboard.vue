<script setup lang="ts">
import { Head } from '@inertiajs/vue3';
import AppLayout from '@/layouts/AppLayout.vue';
import {
    Card,
    CardContent,
    CardDescription,
    CardHeader,
    CardTitle,
} from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
} from '@/components/ui/table';
import {
    Users,
    UserCheck,
    Clock,
    UserPlus,
    ExternalLink,
} from 'lucide-vue-next';
import { dashboard } from '@/routes';
import type { BreadcrumbItem } from '@/types';

interface DashboardProps {
    stats: {
        total_tenants: number;
        active_tenants: number;
        trial_tenants: number;
        new_tenants_this_month: number;
    };
    recentTenants: {
        id: string;
        name: string;
        email: string;
        status: string;
        plan_name: string;
        created_at: string;
        domain_url: string | null;
    }[];
    planDistribution: {
        name: string;
        count: number;
    }[];
}

defineProps<DashboardProps>();

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Dashboard',
        href: dashboard().url,
    },
];

const getStatusVariant = (status: string) => {
    switch (status) {
        case 'Active':
            return 'default';
        case 'Trial':
            return 'secondary';
        case 'Canceled':
            return 'destructive';
        default:
            return 'outline';
    }
};
</script>

<template>
    <Head title="Dashboard" />

    <AppLayout :breadcrumbs="breadcrumbs">
        <div class="flex flex-col gap-6 p-4 md:p-6">
            <!-- KPI Cards -->
            <div class="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
                <Card>
                    <CardHeader
                        class="flex flex-row items-center justify-between space-y-0 pb-2"
                    >
                        <CardTitle class="text-sm font-medium">
                            Total Tenants
                        </CardTitle>
                        <Users class="h-4 w-4 text-muted-foreground" />
                    </CardHeader>
                    <CardContent>
                        <div class="text-2xl font-bold">
                            {{ stats.total_tenants }}
                        </div>
                        <p class="text-xs text-muted-foreground">
                            Registered businesses
                        </p>
                    </CardContent>
                </Card>

                <Card>
                    <CardHeader
                        class="flex flex-row items-center justify-between space-y-0 pb-2"
                    >
                        <CardTitle class="text-sm font-medium">
                            Active Tenants
                        </CardTitle>
                        <UserCheck class="h-4 w-4 text-muted-foreground" />
                    </CardHeader>
                    <CardContent>
                        <div class="text-2xl font-bold">
                            {{ stats.active_tenants }}
                        </div>
                        <p class="text-xs text-muted-foreground">
                            Currently paying
                        </p>
                    </CardContent>
                </Card>

                <Card>
                    <CardHeader
                        class="flex flex-row items-center justify-between space-y-0 pb-2"
                    >
                        <CardTitle class="text-sm font-medium">
                            In Trial
                        </CardTitle>
                        <Clock class="h-4 w-4 text-muted-foreground" />
                    </CardHeader>
                    <CardContent>
                        <div class="text-2xl font-bold">
                            {{ stats.trial_tenants }}
                        </div>
                        <p class="text-xs text-muted-foreground">
                            Potential conversions
                        </p>
                    </CardContent>
                </Card>

                <Card>
                    <CardHeader
                        class="flex flex-row items-center justify-between space-y-0 pb-2"
                    >
                        <CardTitle class="text-sm font-medium">
                            New This Month
                        </CardTitle>
                        <UserPlus class="h-4 w-4 text-muted-foreground" />
                    </CardHeader>
                    <CardContent>
                        <div class="text-2xl font-bold">
                            {{ stats.new_tenants_this_month }}
                        </div>
                        <p class="text-xs text-muted-foreground">Growth rate</p>
                    </CardContent>
                </Card>
            </div>

            <div class="grid gap-4 md:grid-cols-2 lg:grid-cols-7">
                <!-- Recent Tenants List -->
                <Card class="col-span-4">
                    <CardHeader>
                        <CardTitle>Recent Tenants</CardTitle>
                        <CardDescription>
                            Latest 5 registered companies.
                        </CardDescription>
                    </CardHeader>
                    <CardContent>
                        <Table>
                            <TableHeader>
                                <TableRow>
                                    <TableHead>Customer</TableHead>
                                    <TableHead>Plan</TableHead>
                                    <TableHead>Status</TableHead>
                                    <TableHead class="text-right">
                                        Actions
                                    </TableHead>
                                </TableRow>
                            </TableHeader>
                            <TableBody>
                                <TableRow v-if="recentTenants.length === 0">
                                    <TableCell
                                        colspan="4"
                                        class="h-24 text-center text-muted-foreground"
                                    >
                                        No tenants found.
                                    </TableCell>
                                </TableRow>
                                <TableRow
                                    v-for="tenant in recentTenants"
                                    :key="tenant.id"
                                >
                                    <TableCell>
                                        <div class="font-medium">
                                            {{ tenant.name }}
                                        </div>
                                        <div
                                            class="hidden text-sm text-muted-foreground md:inline"
                                        >
                                            {{ tenant.email }}
                                        </div>
                                    </TableCell>
                                    <TableCell>
                                        {{ tenant.plan_name }}
                                    </TableCell>
                                    <TableCell>
                                        <Badge
                                            :variant="
                                                getStatusVariant(tenant.status)
                                            "
                                        >
                                            {{ tenant.status }}
                                        </Badge>
                                    </TableCell>
                                    <TableCell class="text-right">
                                        <a
                                            v-if="tenant.domain_url"
                                            :href="tenant.domain_url"
                                            target="_blank"
                                            rel="noopener noreferrer"
                                            class="inline-flex h-9 w-9 items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors hover:bg-accent hover:text-accent-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 focus-visible:outline-none disabled:pointer-events-none disabled:opacity-50"
                                        >
                                            <ExternalLink class="h-4 w-4" />
                                            <span class="sr-only">Visit</span>
                                        </a>
                                    </TableCell>
                                </TableRow>
                            </TableBody>
                        </Table>
                    </CardContent>
                </Card>

                <!-- Plan Distribution -->
                <Card class="col-span-3">
                    <CardHeader>
                        <CardTitle>Plan Distribution</CardTitle>
                        <CardDescription>
                            Active subscriptions by plan type.
                        </CardDescription>
                    </CardHeader>
                    <CardContent>
                        <div class="space-y-4">
                            <div
                                v-if="planDistribution.length === 0"
                                class="text-sm text-muted-foreground"
                            >
                                No data available.
                            </div>
                            <div
                                v-for="item in planDistribution"
                                :key="item.name"
                                class="flex items-center"
                            >
                                <div
                                    class="flex w-full items-center justify-between"
                                >
                                    <span class="text-sm font-medium">
                                        {{ item.name }}
                                    </span>
                                    <span
                                        class="font-mono text-sm text-muted-foreground"
                                    >
                                        {{ item.count }}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </CardContent>
                </Card>
            </div>
        </div>
    </AppLayout>
</template>
