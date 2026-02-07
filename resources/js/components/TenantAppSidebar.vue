<script setup lang="ts">
import { Link } from '@inertiajs/vue3';
import { Folder, LayoutGrid } from 'lucide-vue-next';
import TenantNavFooter from '@/components/TenantNavFooter.vue';
import TenantNavMain from '@/components/TenantNavMain.vue';
import TenantNavUser from '@/components/TenantNavUser.vue';
import {
    Sidebar,
    SidebarContent,
    SidebarFooter,
    SidebarHeader,
    SidebarMenu,
    SidebarMenuButton,
    SidebarMenuItem,
} from '@/components/ui/sidebar';
import tenant from '@/routes/tenant';
import { type NavItem } from '@/types';
import AppLogo from './AppLogo.vue';

const mainNavItems: NavItem[] = [
    {
        title: 'Dashboard',
        href: tenant.dashboard(),
        icon: LayoutGrid,
    },
];

const footerNavItems: NavItem[] = [
    /*{
        title: 'Repository',
        href: 'https://git.jczap.net/templates/saas-starter-vue',
        icon: Folder,
    },*/
];
</script>

<template>
    <Sidebar collapsible="icon" variant="inset">
        <SidebarHeader>
            <SidebarMenu>
                <SidebarMenuItem>
                    <SidebarMenuButton size="lg" as-child>
                        <Link :href="tenant.dashboard()">
                            <AppLogo />
                        </Link>
                    </SidebarMenuButton>
                </SidebarMenuItem>
            </SidebarMenu>
        </SidebarHeader>

        <SidebarContent>
            <TenantNavMain :items="mainNavItems" />
        </SidebarContent>

        <SidebarFooter>
            <TenantNavFooter :items="footerNavItems" class="mt-auto" />
            <TenantNavUser />
        </SidebarFooter>
    </Sidebar>
    <slot />
</template>
