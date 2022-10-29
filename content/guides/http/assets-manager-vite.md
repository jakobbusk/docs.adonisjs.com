---
summary: Discover how to use the Assets Manager's Vite driver
---

Vite is an amazing frontend build tool that is blazing fast and provides a great developer experience. It is a perfect fit for AdonisJS. This guide will go through the steps to integrate Vite with AdonisJS. 

:::tip
Please refer to the [Vite documentation](https://vitejs.dev/guide/) for more information on how to use Vite.
:::

## Getting Started 

Creating a new AdonisJS application with the Vite driver is as simple as running the following command:

```sh
node ace configure vite
```

This command will :
- Create a basic `vite.config.ts` file in the root of your project.
- Create a `resources/js/app.ts` file. This file will be the entry point of your application.
- Install the required dependencies : 
  - `vite`
  - `@adonisjs/vite-plugin-adonis`: Plugin for vite bridging the gap between AdonisJS and Vite.

## Compiling frontend assets

Once Vite has been installed, you can compile your frontend assets by running the following command:

#### node ace serve --watch

The `node ace serve --watch` command will also run the Vite Dev Server within the same process to compile and serve the frontend assets.

#### node ace build --production

Similarly, the `node ace build --production` command will also run the `vite build` command to bundle the frontend assets alongside your AdonisJS build.

### Customizing output directory

By default, the compiled assets are written to the `./public/assets` directory so that AdonisJS static file server can serve them.

However, you can customize and define any output directory by updating the `vite.config.ts` file.

The `outputPath` property accepts a path relative to the project root. Also, make sure to update the public URL prefix using the `publicPath` property.

```ts
// title: vite.config.ts

import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'

export default defineConfig(({}) => ({
  plugins: [
    Adonis({
      // Write files to this directory
      outputPath: './public/assets',
      
      // Prefix the following to the output URL
      publicPath: '/assets'
    })
  ]
})
```

### Disable assets compilation

You can disable Vite assets compilation by defining the `--no-assets` flag to the `serve` and the `build` commands.

```sh
node ace serve --watch --no-assets
node ace build --productions --no-assets
```

## Customize dev server port and host
Vite dev server runs on `localhost:5173` by default. If the port is in use, a random port will be found to start the Vite dev server. However, you can also define a custom port using the `--assets-bundler-args` flag.

```sh
node ace serve --watch --assets-bundler-args="--port 5000"
```

You can also define a custom port within the `vite.config.ts` file. See the [Vite docs](https://vitejs.dev/config/server-options.html#server-port) for more details.


## Assets view helpers

When you build your frontend assets, Vite will add a hash to the compiled files. This is done to avoid browser caching issues.

Hence, it is recommended to never hardcode the file names in your templates and always use the `asset` helper.

:::caption{for="error"}
Do not reference files by name
:::

```edge
<!DOCTYPE html>
<html lang="en">
<head>
    // highlight-start
  <script src="/assets/app.js"></script>
  <link rel="stylesheet" type="text/css" href="/assets/app.css">  
    // highlight-end
</head>
<body>
</body>
</html>
```

:::caption{for="success"}
Use the `asset` helper
:::

```edge
<!DOCTYPE html>
<html lang="en">
<head>
  // highlight-start
  <script src="{{ asset('assets/app.js') }}"></script>
  <link rel="stylesheet" type="text/css" href="{{ asset('assets/app.css') }}">  
    // highlight-end
</head>
<body>
</body>
</html>
```

The `asset` helper relies on the `manifest.json` file generated by the Encore to resolve the actual URL. You can use it for all the assets, including JavaScript, CSS, fonts, images, and so on.

## Manifest file

When building for production, Vite will generates the `manifest.json` file inside the `public/assets` directory. This file contains all the compiled assets with some additional information, including the hashed file name.

Here is an example :

```json
{
  "resources/js/app.ts": {
    "file": "app.3657b05e.js",
    "src": "resources/js/app.ts",
    "isEntry": true,
    "imports": [
      "_runtime-dom.esm-bundler.6ec352c0.js",
      "_ace.a1f217ec.js"
    ],
    "dynamicImports": [
      "_ace.a1f217ec.js"
    ],
    "css": [
      "app.d90c71c1.css"
    ]
  },
  "resources/css/app.css": {
    "file": "app.2b8046fe.css",
    "src": "resources/css/app.css",
    "isEntry": true
  },
  "resources/js/app.css": {
    "file": "app.d90c71c1.css",
    "src": "resources/js/app.css"
  }
}
```

The `asset` view helper resolves the URL from this file itself.

:::tip
Note that the `manifest.json` file is only generated when building for production. It is not generated when running the `node ace serve` command.
:::

## Entrypoints

Every Vite bundle always has one or more entrypoints. Any other imports inside the entry point file are part of the same bundle. 

For example, if you have registered the `./resources/js/app.js` file as an entry point with the following contents, all the internal imports will be bundled together to form a single output.

```ts
import '../css/app.css'
import 'normalize.css'
import 'alpinejs'
```

You can define these entry points inside the `vite.config.js` file using as follows:

```ts
// title: vite.config.ts

import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'

export default defineConfig(({}) => ({
  plugins: [
    Adonis({
      entryPoints: {
        app: ['./resources/js/app.ts'],
      }
    })
  ]
})
```

### Multiple entry points

Most applications need a single entry point unless you are building multiple interfaces in a single codebase. For example: Creating a public website + an admin panel may require different entry points as they will usually have different frontend dependencies and styling altogether.

You can define multiple entry points by just adding more keys to the `entryPoints` object.

### Reference entry points inside the template files

You can make use of the `@entryPointStyles` and the `@entryPointScripts` tags to render the script and the style tags for a given entry point.

The tags will output the HTML with the correct `href` and `src` attributes. The `./public/assets/entrypoints.json` file is used to look up the URLs for a given entry point.

```edge
<!DOCTYPE html>
<html lang="en">
<head>
  @entryPointScripts('app')
  @entryPointStyles('app')
</head>
<body>
</body>
</html>
```

## Referencing images, fonts, and other assets

Vite cannot automatically scan/process the assets referenced inside an Edge template. Hence, you have to tell the Vite in advance to process the assets from a specific directory.

You can use the `import.meta.glob` method to instruct Vite which files to bundle.

```ts
// title: resources/js/app.ts

import.meta.glob([
  '../images/**', 
  '../fonts/**'
])
```

Now you can reference assets with the `asset` tag

```edge
<img src="{{ asset('assets/images/logo.png') }}" />
```

## Configuring HMR

To enable HMR, make sure to always add the `@vite` tag to the top of your edge templates : 

```edge
<!DOCTYPE html>
<html lang="en">
<head>
  // highlight-start
  @vite()
  // highlight-end
  @entryPointScripts('app')
  @entryPointStyles('app')
</head>
<body>
</body>
</html>
```

## Configuring Vue

You can configure Vue using the `@vitejs/plugin-vue` plugin. 

```ts
// title: vite.config.ts

import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'
import Vue from '@vitejs/plugin-vue'

export default defineConfig(({}) => ({
  plugins: [
    Adonis({
      // ...
    }),
    Vue()
  ]
})
```

## Configuring React

You can configure React using the `@vitejs/plugin-react` plugin. 

```ts
// title: vite.config.ts

import { defineConfig } from 'vite'
import Adonis from '@adonisjs/vite-plugin-adonis'
import React from '@vitejs/plugin-react'

export default defineConfig(({}) => ({
  plugins: [
    Adonis({
      // ...
    }),
    React()
  ]
})
```

Also make sure to always add the `@viteReactRefresh` tag ( along with the `@vite` tag ) to the top of your edge templates : 

```edge
<!DOCTYPE html>
<html lang="en">
<head>
  // highlight-start
  @vite()
  @viteReactRefresh()
  // highlight-end
  @entryPointScripts('app')
  @entryPointStyles('app')
</head>
<body>
</body>
</html>
```

## Inertia

If you are using Inertia with [Lev Eidelman Nagar](https://github.com/eidellev)'s awesome [`@eidellev/inertia-adonisjs`](https://github.com/eidellev/inertiajs-adonisjs) package, you can use the following helper to resolve your components : 

```ts
import { createApp, h } from 'vue'
import { createInertiaApp } from '@inertiajs/inertia-vue3'
// insert-start
+ import { resolvePageComponent } from '@adonisjs/vite-plugin-adonis/inertia'
// insert-end
createInertiaApp({
  // delete-start
  resolve: (name) => require(`./Pages/${name}`),
  // delete-end
  // insert-start
  resolve: (name) => {
    return resolvePageComponent(
      `./Pages/${name}.vue`, 
      import.meta.glob('./Pages/**/*.vue')
    )
  },
  // insert-end
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
})
```

The above example is for Vue.js, but with React or other frameworks, the process is almost the same.