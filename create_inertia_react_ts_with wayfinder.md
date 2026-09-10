## 1. Server-side: Inertia adapter

```bash
composer require inertiajs/inertia-laravel
```

Publish the middleware:

```bash
php artisan inertia:middleware
```

This creates `app/Http/Middleware/HandleInertiaRequests.php`.

## 2. Register the middleware

In `app.php`, add `use App\Http\Middleware\HandleInertiaRequests;` at the top and append it inside `withMiddleware`:

```php
use App\Http\Middleware\HandleInertiaRequests;

return Application::configure(basePath: dirname(__DIR__))
    // ...
    ->withMiddleware(function (Middleware $middleware): void {
        $middleware->web(append: [
            HandleInertiaRequests::class,
        ]);
    })
    // ...
    ->create();
```

## 3. Create the root template

Create `resources/views/app.blade.php`:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        @viteReactRefresh
        @vite(['resources/css/app.css', 'resources/js/app.tsx'])
        <x-inertia::head />
    </head>
    <body class="font-sans antialiased">
        <x-inertia::app />
    </body>
</html>
```

## 4. Client-side: install React + TypeScript deps

```bash
npm install @inertiajs/react react react-dom
npm install -D @vitejs/plugin-react typescript @types/react @types/react-dom
```

## 5. Add `tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "ES2020",
        "useDefineForClassFields": true,
        "lib": ["ES2020", "DOM", "DOM.Iterable"],
        "module": "ESNext",
        "skipLibCheck": true,
        "moduleResolution": "bundler",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "react-jsx",
        "strict": true,
        "paths": {
            "@/*": ["resources/js/*"]
        },
        "types": ["vite/client"]
    },
    "include": [
        "resources/js/**/*.ts",
        "resources/js/**/*.tsx",
        "resources/js/**/*.d.ts"
    ]
}
```

The `@/*` alias is what Wayfinder's generated imports (`@/actions/...`, `@/routes/...`) rely on.

## 6. Replace `app.js` with `app.tsx`

Delete `app.js` and create `resources/js/app.tsx`:

```tsx
import { createInertiaApp } from '@inertiajs/react'
import { createRoot } from 'react-dom/client'
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers'

createInertiaApp({
  resolve: (name) =>
    resolvePageComponent(
      `./Pages/${name}.tsx`,
      import.meta.glob('./Pages/**/*.tsx'),
    ),
  setup({ el, App, props }) {
    createRoot(el).render(<App {...props} />)
  },
})
```

## 7. Rename and update the Vite config

Rename `vite.config.js` → `vite.config.ts`:

```ts
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import { bunny } from 'laravel-vite-plugin/fonts'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import { wayfinder } from '@laravel/vite-plugin-wayfinder'
import path from 'node:path'

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.tsx'],
            refresh: true,
            fonts: [
                bunny('Instrument Sans', {
                    weights: [400, 500, 600],
                }),
            ],
        }),
        react(),
        tailwindcss(),
        wayfinder(),
    ],
    resolve: {
        alias: {
            "@": `${import.meta.dirname}/resources/js`,
        },
    },
    server: {
        watch: {
            ignored: ['**/storage/framework/views/**'],
        },
    },
})
```

## 8. Install Wayfinder (server + Vite plugin)

```bash
composer require laravel/wayfinder
npm install -D @laravel/vite-plugin-wayfinder
```

Then generate the TypeScript definitions once:

```bash
php artisan wayfinder:generate
```

This creates `resources/js/wayfinder/`, `resources/js/actions/`, and `resources/js/routes/`. You can add those three directories to `.gitignore` since they're fully regenerated.

## 9. Wire up your first page

Controller/route (`web.php`):

```php
use Inertia\Inertia;

Route::get('/', fn () => Inertia::render('Home'));
```

Page component (`resources/js/Pages/Home.tsx`):

```tsx
export default function Home() {
  return <h1 className="text-2xl font-semibold">Hello, Inertia + React!</h1>
}
```

## 10. Run it

```bash
composer run dev
```

or separately: `php artisan serve` + `npm run dev`.

## Wayfinder usage in React

With an action on `PostController@show` and named route `post.show`:

```tsx
import { Link, useForm } from '@inertiajs/react'
import { show } from '@/actions/App/Http/Controllers/PostController'

// Link to a route
<Link href={show(1)}>Post #1</Link>          // { url: "/posts/1", method: "get" }

// URL only
show.url(1)                                   // "/posts/1"

// Named route
import { show as postShow } from '@/routes/post'
postShow(1)

// Forms
const form = useForm({ name: 'New post' })
form.submit(store())                          // auto-resolves URL + method
```

