<script setup lang="ts">
import { Head, Link } from '@inertiajs/vue3';
import AuthBase from '@/layouts/AuthLayout.vue';
import { Button } from '@/components/ui/button';
</script>

<template>
    <Head title="Registro deshabilitado" />

    <AuthBase
        title="Registro deshabilitado"
        description="El registro de invitados está actualmente deshabilitado."
    >
        <div
            class="flex flex-col gap-4 rounded-lg border border-muted bg-muted/30 p-4 text-center"
        >
            <p class="text-sm text-muted-foreground">
                No es posible crear una cuenta en este momento. Si necesitas
                acceso, contacta al administrador del sistema.
            </p>
            <!-- Uncomment if needed
            <Button as-child variant="default" class="w-full">
                <Link :href="route('login')">Ir a iniciar sesión</Link>
            </Button>
            -->
        </div>
    </AuthBase>
</template>
