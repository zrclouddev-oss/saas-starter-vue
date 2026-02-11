# SaaS Starter Vue

Este proyecto es una plantilla robusta y moderna para iniciar aplicaciones SaaS (Software as a Service) rápidamente. Está construido con **Laravel 12** en el backend y **Vue 3** (vía Inertia.js 2.0) en el frontend, utilizando **Tailwind CSS 4** para el diseño.

## 🚀 Características Principales

-   **Multi-tenancy**: Gestión completa de inquilinos (tenants) utilizando `stancl/tenancy`.
-   **Gestión de Planes**: Sistema integrado para crear y administrar planes de suscripción.
-   **Autenticación Robusta**: Implementada con Laravel Fortify, incluye registro, login, recuperación de contraseña y **Autenticación de Dos Factores (2FA)**.
-   **Frontend Moderno**: SPA monolítica con Vue 3, Inertia.js 2.0 y TypeScript.
-   **Diseño**: Estilizado con Tailwind CSS 4 y componentes UI reutilizables (`reka-ui`, `lucide-vue-next`).
-   **Base de Datos**: Configurado por defecto para PostgreSQL.
-   **Gestión de Paquetes**: Utiliza PNPM para una gestión eficiente de dependencias de frontend.
-   **Correo**: Configuración lista para SMTP.

## 🛠️ Stack Tecnológico

-   **Backend**: Laravel 12, PHP 8.2+
-   **Frontend**: Vue 3, Inertia.js 2, TypeScript
-   **Estilos**: Tailwind CSS 4
-   **Base de Datos**: PostgreSQL
-   **Herramientas**: Vite, PNPM, Composer

## 📋 Requisitos Previos

Asegúrate de tener instalados los siguientes componentes en tu entorno de desarrollo:

-   PHP >= 8.2
-   Composer
-   Node.js & PNPM
-   PostgreSQL

## ⚡ Instalación y Configuración

Sigue estos pasos para levantar el proyecto en tu entorno local:

1.  **Clonar el repositorio**:
    ```bash
    git clone https://github.com/zrclouddev-oss/saas-starter-vue.git
    cd saas-starter-vue
    ```

2.  **Configurar Variables de Entorno**:
    Copia el archivo de ejemplo y configura tu base de datos y otros servicios.
    ```bash
    cp .env.example .env
    ```
    Asegúrate de configurar correctamente la conexión a la base de datos en el archivo `.env`:
    ```ini
    DB_CONNECTION=pgsql
    DB_HOST=127.0.0.1
    DB_PORT=5432
    DB_DATABASE=nombre_de_tu_bd
    DB_USERNAME=tu_usuario
    DB_PASSWORD=tu_password
    ```

3.  **Instalación Automática**:
    El proyecto incluye un script de `composer` que instala dependencias de backend y frontend, genera la key de la aplicación y corre las migraciones.
    ```bash
    composer run setup
    ```
    *Este comando ejecuta: `composer install`, copia `.env` (si no existe), `key:generate`, `migrate --force`, `pnpm install` y `pnpm run build`.*

## ▶️ Ejecución

Para iniciar el entorno de desarrollo, utiliza el siguiente comando que levantará el servidor de Laravel, las colas y Vite concurrentemente:

```bash
composer run dev
```

El sitio estará disponible en la URL configurada (por defecto `http://saas-starter-vue.test` o `http://localhost:8000` si usas `php artisan serve` directamente, pero el comando `dev` usa `php artisan serve`).

## 🧪 Tests y Calidad de Código

-   **Linting (Backend)**:
    ```bash
    composer run lint
    ```
-   **Tests**:
    ```bash
    composer run test
    ```
-   **Formato (Frontend)**:
    ```bash
    pnpm run format
    ```

## 📂 Estructura del Proyecto

-   `/app`: Lógica del backend (Modelos, Controladores, Tenancy).
-   `/resources/js`: Código fuente del frontend (Vue, Componentes, Pages).
-   `/routes`: Definición de rutas (Web, API, Auth).
-   `/database`: Migraciones y Seeders.

---
Creado por [ZRCloud](https://git.jczap.net/cloud).
