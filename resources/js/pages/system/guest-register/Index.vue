<script setup lang="ts">
import { ref } from 'vue';
import { Head, useForm } from '@inertiajs/vue3';
import AuthBase from '@/layouts/AuthLayout.vue';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import InputError from '@/components/InputError.vue';

interface FreePlan {
    id: number;
    name: string;
    description?: string;
}

interface IndexProps {
    app_url_base: string;
    free_plan: FreePlan | null;
}

const props = defineProps<IndexProps>();

const form = useForm({
    company_name: '',
    owner_name: '',
    owner_email: '',
    owner_phone: '',
    domain: '',
    password: '',
    password_confirmation: '',
});

const submit = () => {
    form.post('/guest-register', {
        onFinish: () => form.reset('password', 'password_confirmation'),
    });
};

// For existing users to access their tenant
const loginSubdomain = ref('');

const goToTenantLogin = () => {
    const subdomain = loginSubdomain.value.trim();
    if (!subdomain) return;

    const tenantUrl = `${window.location.protocol}//${subdomain}.${props.app_url_base}/login`;
    window.location.href = tenantUrl;
};

const handleLoginSubdomainKeyPress = (e: KeyboardEvent) => {
    if (e.key === 'Enter') {
        goToTenantLogin();
    }
};
</script>

<template>
    <Head title="Registro de invitados" />

    <AuthBase
        title="Crear tu cuenta"
        description="Completa el formulario para registrar tu empresa y acceder a la plataforma."
    >
        <p
            v-if="free_plan"
            class="mb-4 rounded-lg border border-muted bg-muted/30 px-3 py-2 text-sm text-muted-foreground"
        >
            Se te asignará el plan <strong>{{ free_plan.name }}</strong> al
            registrarte.
        </p>

        <form @submit.prevent="submit" class="flex flex-col gap-4">
            <div class="grid gap-2">
                <Label for="company_name">Nombre de la empresa</Label>
                <Input
                    id="company_name"
                    v-model="form.company_name"
                    type="text"
                    required
                    autofocus
                    placeholder="Mi Empresa S.A.C."
                />
                <InputError :message="form.errors.company_name" />
            </div>

            <div class="grid gap-2">
                <Label for="owner_name">Nombre del responsable</Label>
                <Input
                    id="owner_name"
                    v-model="form.owner_name"
                    type="text"
                    required
                    placeholder="Juan Pérez"
                />
                <InputError :message="form.errors.owner_name" />
            </div>

            <div class="grid gap-2">
                <Label for="owner_email">Correo electrónico</Label>
                <Input
                    id="owner_email"
                    v-model="form.owner_email"
                    type="email"
                    required
                    placeholder="juan@miempresa.com"
                />
                <InputError :message="form.errors.owner_email" />
            </div>

            <div class="grid gap-2">
                <Label for="domain">Subdominio</Label>
                <div class="flex items-center gap-1">
                    <Input
                        id="domain"
                        v-model="form.domain"
                        type="text"
                        required
                        placeholder="miempresa"
                        class="rounded-r-none"
                    />
                    <span
                        class="rounded-r-md border border-input bg-muted px-3 py-2 text-sm text-muted-foreground"
                    >
                        .{{ app_url_base }}
                    </span>
                </div>
                <p class="text-xs text-muted-foreground">
                    Solo letras, números y guiones. Mínimo 3 caracteres.
                </p>
                <InputError :message="form.errors.domain" />
            </div>

            <div class="grid gap-2">
                <Label for="password">Contraseña</Label>
                <Input
                    id="password"
                    v-model="form.password"
                    type="password"
                    required
                    placeholder="Mínimo 8 caracteres"
                />
                <InputError :message="form.errors.password" />
            </div>

            <div class="grid gap-2">
                <Label for="password_confirmation">Confirmar contraseña</Label>
                <Input
                    id="password_confirmation"
                    v-model="form.password_confirmation"
                    type="password"
                    required
                    placeholder="Repite la contraseña"
                />
                <InputError :message="form.errors.password_confirmation" />
            </div>

            <InputError :message="(form.errors as any).plan" />
            <InputError :message="(form.errors as any).submit" />

            <Button type="submit" class="w-full" :disabled="form.processing">
                {{ form.processing ? 'Creando cuenta...' : 'Crear cuenta' }}
            </Button>
        </form>

        <!-- Login section for existing users -->
        <div class="mt-6 pt-6 border-t border-muted">
            <p class="text-center text-sm text-muted-foreground mb-3">
                ¿Ya tienes cuenta? Ingresa tu subdominio para iniciar sesión:
            </p>
            <div class="flex gap-2">
                <div
                    class="flex-1 flex items-center gap-1 rounded-md border border-input bg-background px-3 py-2 text-sm"
                >
                    <input
                        v-model="loginSubdomain"
                        type="text"
                        placeholder="tu-empresa"
                        class="flex-1 bg-transparent outline-none placeholder:text-muted-foreground"
                        @keyup="handleLoginSubdomainKeyPress"
                    />
                    <span class="text-muted-foreground"
                        >.{{ app_url_base }}</span
                    >
                </div>
                <Button
                    type="button"
                    variant="outline"
                    @click="goToTenantLogin"
                    :disabled="!loginSubdomain.trim()"
                >
                    Ir
                </Button>
            </div>
        </div>
    </AuthBase>
</template>
