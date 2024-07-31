# How to setup project

-   Installation
-   Setup Inertia JS
-   Inertia JS Server Side
-   Inertia JS Client Side
-   Setup Vite React Plugin

## Installation

install fresh new laravel project

```bash
laravel new project_name
```

## Setup Inertia JS

Inertia is a new approach to building classic server-driven web apps. We call it the modern monolith.

## Inertia JS Server Side

### Install dependencies

First, install the Inertia server-side adapter using the Composer package manager.

```bash
composer require inertiajs/inertia-laravel
composer require tightenco/ziggy
```

### Root template

Next, setup the root template that will be loaded on the first page visit to your application. This will be used to load your site assets (CSS and JavaScript), and will also contain a root <div> in which to boot your JavaScript application.

```code
<!-- resource/view/app.blade.php-->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    <!-- Scripts and Styles -->
    @routes
    @viteReactRefresh
    @vite('resources/js/app.tsx')
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

### Middleware

```bash
php artisan inertia:middleware
```

```code
// app/http/kernel.php
'web' => [
    // ...
    \App\Http\Middleware\HandleInertiaRequests::class,
],

// laravel 11.*
use App\Http\Middleware\HandleInertiaRequests;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        HandleInertiaRequests::class,
    ]);
})
```

## Inertia JS Client Side

### Install dependencies

```bash
npm install @inertiajs/react react react-dom
```

### Install TypeScript and Necessary Packages

Install TypeScript and other necessary packages:

```bash
npm install typescript @types/node @types/react @types/react-dom
```

### Configure TypeScript

Initialize TypeScript configuration:

```code
npx tsc --init
```

Then, you need to edit tsconfig.json to suit your project setup. Here is a simple example configuration:

```code
// tsconfig.json
{
    "compilerOptions": {
        "allowJs": true,
        "module": "ESNext",
        "moduleResolution": "bundler",
        "jsx": "react-jsx",
        "strict": true,
        "isolatedModules": true,
        "target": "ESNext",
        "esModuleInterop": true,
        "forceConsistentCasingInFileNames": true,
        "noEmit": true,
        "paths": {
            "@/*": ["./resources/js/*"],
            "ziggy-js": ["./vendor/tightenco/ziggy"]
        }
    },
    "include": ["resources/js/**/*.ts", "resources/js/**/*.tsx", "resources/js/**/*.d.ts"]
}


```

### Initialize the Inertia app

resource/js/app.tsx

```code
import { createInertiaApp } from '@inertiajs/react'
import { createRoot } from 'react-dom/client'

createInertiaApp({
  resolve: name => {
    //@ts-ignore
    const pages = import.meta.glob('./Pages/**/*.tsx', { eager: true })
    return pages[`./Pages/${name}.tsx`]
  },
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />)
  },
})
```

## Setup Vite React Plugin

```bash
npm i @vitejs/plugin-react @types/vite
```

```code
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.tsx',
            refresh: true,
        }),
        react(),
    ],
});


```
