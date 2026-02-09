<script setup lang="ts">
import { Head, router } from '@inertiajs/vue3';
import { Plus, Pencil, Trash2, Folder } from 'lucide-vue-next';
import { ref } from 'vue';
import Heading from '@/components/Heading.vue';
import { Button } from '@/components/ui/button';
import {
    Table,
    TableBody,
    TableCell,
    TableHead,
    TableHeader,
    TableRow,
} from '@/components/ui/table';
import TenantAppLayout from '@/layouts/TenantAppLayout.vue';
import type { BreadcrumbItem } from '@/types';
import UserFormModal from './components/UserFormModal.vue';
import tenant from '@/routes/tenant';

const breadcrumbs: BreadcrumbItem[] = [
    {
        title: 'Users',
        href: '/users',
    },
];

interface User {
    id: number;
    name: string;
    email: string;
    created_at: string;
}

interface Props {
    users: {
        data: User[];
        links: any[];
        meta: any;
    };
}

const props = defineProps<Props>();

const showModal = ref(false);
const selectedUser = ref<User | null>(null);

const openCreateModal = () => {
    selectedUser.value = null;
    showModal.value = true;
};

const openEditModal = (user: User) => {
    selectedUser.value = user;
    showModal.value = true;
};

const closeModal = () => {
    showModal.value = false;
    selectedUser.value = null;
};

const deleteUser = (user: User) => {
    if (confirm(`Are you sure you want to delete ${user.name}?`)) {
        router.delete(`/users/${user.id}`);
    }
};

const formatDate = (dateString: string) => {
    return new Date(dateString).toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
    });
};
</script>

<template>
    <Head title="Users" />

    <TenantAppLayout :breadcrumbs="breadcrumbs">
        <div
            class="flex h-full flex-1 flex-col gap-4 overflow-x-auto rounded-xl p-4"
        >
            <!-- Header with Action -->
            <div class="flex items-center justify-between">
                <Heading
                    title="Users"
                    description="Manage users who have access to this tenant."
                />
                <Button @click="openCreateModal">
                    <Plus class="mr-2 h-4 w-4" />
                    Add User
                </Button>
            </div>

            <!-- Statistics Cards -->
            <div class="grid auto-rows-min gap-4 md:grid-cols-3">
                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 shadow-sm dark:border-sidebar-border"
                >
                    <div class="flex items-center justify-between space-x-4">
                        <div class="flex-1">
                            <p class="text-sm font-medium text-muted-foreground">
                                Total Users
                            </p>
                            <p class="mt-2 text-3xl font-bold">
                                {{ users?.meta?.total || 0 }}
                            </p>
                        </div>
                        <div class="rounded-full bg-primary/10 p-3">
                            <Folder class="h-6 w-6 text-primary" />
                        </div>
                    </div>
                </div>

                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 shadow-sm dark:border-sidebar-border"
                >
                    <div class="flex items-center justify-between space-x-4">
                        <div class="flex-1">
                            <p class="text-sm font-medium text-muted-foreground">
                                Active Today
                            </p>
                            <p class="mt-2 text-3xl font-bold">
                                {{ users?.data?.length || 0 }}
                            </p>
                        </div>
                        <div class="rounded-full bg-green-500/10 p-3">
                            <Folder class="h-6 w-6 text-green-500" />
                        </div>
                    </div>
                </div>

                <div
                    class="relative overflow-hidden rounded-xl border border-sidebar-border/70 bg-card p-6 shadow-sm dark:border-sidebar-border"
                >
                    <div class="flex items-center justify-between space-x-4">
                        <div class="flex-1">
                            <p class="text-sm font-medium text-muted-foreground">
                                New This Month
                            </p>
                            <p class="mt-2 text-3xl font-bold">
                                {{ users?.data?.length || 0 }}
                            </p>
                        </div>
                        <div class="rounded-full bg-blue-500/10 p-3">
                            <Folder class="h-6 w-6 text-blue-500" />
                        </div>
                    </div>
                </div>
            </div>

            <!-- Users Table Card -->
            <div
                class="relative flex-1 rounded-xl border border-sidebar-border/70 bg-card dark:border-sidebar-border"
            >
                <div class="p-6">
                    <h3 class="mb-4 text-lg font-semibold">All Users</h3>
                    <Table>
                        <TableHeader>
                            <TableRow>
                                <TableHead>Name</TableHead>
                                <TableHead>Email</TableHead>
                                <TableHead>Created</TableHead>
                                <TableHead class="text-right">Actions</TableHead>
                            </TableRow>
                        </TableHeader>
                        <TableBody>
                            <TableRow v-if="users.data.length === 0">
                                <TableCell
                                    colspan="4"
                                    class="py-8 text-center text-muted-foreground"
                                >
                                    No users found. Create your first user to get
                                    started.
                                </TableCell>
                            </TableRow>
                            <TableRow v-for="user in users.data" :key="user.id">
                                <TableCell class="font-medium">{{
                                    user.name
                                }}</TableCell>
                                <TableCell>{{ user.email }}</TableCell>
                                <TableCell>{{
                                    formatDate(user.created_at)
                                }}</TableCell>
                                <TableCell class="text-right">
                                    <div class="flex justify-end gap-2">
                                        <Button
                                            variant="ghost"
                                            size="sm"
                                            @click="openEditModal(user)"
                                        >
                                            <Pencil class="h-4 w-4" />
                                        </Button>
                                        <Button
                                            variant="ghost"
                                            size="sm"
                                            @click="deleteUser(user)"
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
                <div
                    v-if="users?.meta?.last_page > 1"
                    class="flex justify-center gap-2 border-t p-4"
                >
                    <Button
                        v-for="link in users.links"
                        :key="link.label"
                        :variant="link.active ? 'default' : 'outline'"
                        size="sm"
                        :disabled="!link.url"
                        @click="link.url && router.visit(link.url)"
                        v-html="link.label"
                    />
                </div>
            </div>
        </div>

        <!-- User Form Modal -->
        <UserFormModal
            :show="showModal"
            :user="selectedUser"
            @close="closeModal"
        />
    </TenantAppLayout>
</template>
