<a href="/../../#readme">
  <img src="/docs/logo.svg" align="right" height="160" alt="vite-plugin-ssr"/>
</a>

# `vite-plugin-ssr`

Do-One-Thing-Do-It-Well, Flexible, Simple.

<a href="https://discord.gg/qTq92FQzKb"><img src="/docs/discord.svg" height="30" alt="vite-plugin-ssr discord"/></a>

[Introduction](#introduction)
<br/> [Vue Tour](#vue-tour)
<br/> [React Tour](#react-tour)
<br/> Get Started
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Boilerplates](#boilerplates)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Manual Installation](#manual-installation)
<br/> Guides
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Data Fetching](#data-fetching)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Routing](#routing)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Auth Data](#auth-data)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [HTML `<head>`](#html-head)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Store](#store)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Markdown](#markdown)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Pre-rendering](#pre-rendering)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Page Redirection](#page-redirection)
<br/> API
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`*.page.js`](#pagejs)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`*.page.client.js`](#pageclientjs)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`import { getPage } from 'vite-plugin-ssr/client'`](#import--getpage--from-vite-plugin-ssrclient)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`*.page.route.js`](#pageroutejs)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Route String](#route-string)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Route Function](#route-function)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`*.page.server.js`](#pageserverjs)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`export { addContextProps }`](#export--addcontextprops-)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`export { setPageProps }`](#export--setpageprops-)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`export { prerender }`](#export--prerender-)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`export { render }`](#export--render-)
<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`import { html } from 'vite-plugin-ssr'`](#import--html--from-vite-plugin-ssr)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`_default.*`](#_default)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`_404.page.js`](#_404pagejs)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`_500.page.js`](#_500pagejs)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [Filesystem Routing](#filesystem-routing)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`import { createRender } from 'vite-plugin-ssr'`](#import--createrender--from-vite-plugin-ssr)
<br/> &nbsp;&nbsp;&nbsp;&#8226;&nbsp; [`import vitePlugin from 'vite-plugin-ssr'`](#import-viteplugin-from-vite-plugin-ssr)

<br/>


## Introduction

`vite-plugin-ssr` provides a similar experience than Nuxt/Next.js, but with Vite's wonderful DX, and as a do-one-thing-do-it-well tool.

- **Do-One-Thing-Do-It-Well**. Only takes care of SSR and works with: other Vite plugins, any view framework (Vue 3, Vue 2, React, Svelte, Preact, Solid, ...), and any server framework (Express, Koa, Hapi, Fastify, ...).
- **Render Control**. You control how your pages are rendered enabling you to easily and naturally integrate tools (Vuex, Redux, Apollo GraphQL, Service Workers, ...).
- **Routing**. Supports Filesystem Routing for basic needs, Route Strings for simple parameterized routes, Route Functions for full flexibility, and can be used with Vue Router or React Router for client-side dynamic nested routes.
- **Pre-render / SSG / Static Websites**. Deploy your app to a static host by pre-rendering your pages.
- **Scalable**. Thanks to Vite's lazy transpiling, Vite apps can scale to thousands of modules with no hit on dev speed.
- **Fast Production Cold Start**. Your pages' server-side code is lazy loaded so that adding pages doesn't increase cold start.
- **Code Splitting**. Each page loads only the browser-side code it needs.
- **Simple Design**. Simple overall design resulting in a tool that is small, robust, and easy to use.

To get an idea of what it's like to use `vite-plugin-ssr`, checkout the [Vue Tour](#vue-tour) or [React Tour](#react-tour).

<br/><br/>


## Vue Tour

Similarly to SSR frameworks,
pages are defined by page files.

```vue
<!-- /pages/index.page.vue -->
<!-- Environment: Browser, Node.js -->

<template>
  This page is rendered to HTML and interactive:
  <button @click="state.count++">Counter {{ state.count }}</button>
</template>

<script>
import { reactive } from 'vue'
export default {
  setup() {
    const state = reactive({ count: 0 })
    return { state }
  }
}
</script>
```

By default, `vite-plugin-ssr` does filesystem routing:
```
FILESYSTEM                  URL
pages/index.page.vue        /
pages/about.page.vue        /about
```

You can also use Route Strings (for parameterized routes such as `/movies/:id`) and Route Functions (for full programmatic flexibility).

```js
// /pages/index.page.route.js
// Environment: Node.js

export default '/'
```

Unlike SSR frameworks,
*you* define how your pages are rendered.

```js
// /pages/_default.page.server.js
// Environment: Node.js

import { createSSRApp, h } from 'vue'
import { renderToString } from '@vue/server-renderer'
import { html } from 'vite-plugin-ssr'

export { render }

async function render({ Page, pageProps }) {
  const app = createSSRApp({
    render: () => h(Page, pageProps)
  })

  const appHtml = await renderToString(app)

  return html`<!DOCTYPE html>
    <html>
      <head>
        <title>Vite w/ SSR Demo</title>
      </head>
      <body>
        <div id="app">${html.dangerouslySetHtml(appHtml)}</div>
      </body>
    </html>`
}
```
```js
// /pages/_default.page.client.js
// Environment: Browser

import { createSSRApp, h } from 'vue'
import { getPage } from 'vite-plugin-ssr/client'

hydrate()

async function hydrate() {
  // (In production, the page is `<link rel="preload">`'d.)
  const { Page, pageProps } = await getPage()

  const app = createSSRApp({
    render: () => h(Page, pageProps)
  })

  app.mount('#app')
}
```

The `render()` hook in `pages/_default.page.server.js` gives you full control over how your pages are rendered,
and `pages/_default.page.client.js` gives you full control over the browser-side code.
This control enables you to *easily* and *naturally*:
 - Use any tool you want such as Vue Router and Vuex.
 - Use any Vue version you want.

Note how the files we created so far end with `.page.vue`, `.page.route.js`, `.page.server.js`, and `.page.client.js`.
 - `.page.js`: defines the page's view that is rendered to HTML / the DOM.
 - `.page.client.js`: defines the page's browser-side code.
 - `.page.server.js`: defines the page's hooks (always run in Node.js).
 - `.page.route.js`: defines the page's Route String or Route function.

Using `vite-plugin-ssr` consists simply of writing these four types of files.

Instead of creating a `.page.client.js` and `.page.server.js` file for each page, you can create `_default.page.client.js` and `_default.page.server.js` which apply as default for all pages.

We already defined our `_default.*` files above,
which means that we can now create a new page simply by defining a new `.page.vue` file.
(The `.page.route.js` file is optional and only needed if we want to define a parameterized route.)

The `_default.*` files can be overwriten. For example, you can create a page with a different browser-side code than your other pages.

```js
// /pages/about.page.client.js

// This file is empty which means that the `/about` page has zero browser-side JavaScript.
```
```vue
<!-- /pages/about.page.vue -->

<template>
  This page is only rendered to HTML.
</template>
```

By overwriting `_default.page.server.js` you can
even render some of your pages with an entire different view framework such as React.

Note how files are collocated and share the same base `/pages/about.page.*`;
this is how you tell `vite-plugin-ssr` that `/pages/about.page.client.js` is the browser-side code of `/pages/about.page.vue`.

Let's now have a look at how to fetch data for a page that has a parameterized route.

```vue
<!-- /pages/star-wars/movie.page.vue -->
<!-- Environment: Browser, Node.js -->

<template>
  <h1>{{movie.title}}</h1>
  <p>Release Date: {{movie.release_date}}</p>
  <p>Director: {{movie.director}}</p>
</template>

<script lang="ts">
const pageProps = ['movie']
export default { props: pageProps }
</script>
```
```js
// /pages/star-wars/movie.page.route.js
// Environment: Node.js

export default '/star-wars/:movieId'
```
```js
// /pages/star-wars/movie.page.server.js
// Environment: Node.js

import fetch from 'node-fetch'

export { addContextProps }
export { setPageProps }

async function addContextProps({ contextProps }) {
  // Route parameters are available at `contextProps`
  const { movieId } = contextProps
  // We could also use SQL/ORM queries here
  const response = await fetch(`https://swapi.dev/api/films/${movieId}`)
  const movie = await response.json()
  return { movie }
}

// The `contextProps` are available only on the server, and only the `pageProps` are
// serialized and passed to the browser.
function setPageProps({ contextProps }) {
  // We select only the data we need in order to minimize what it sent over the network
  const { title, release_date, director } = contextProps.movie
  const movie = { title, release_date, director }
  const pageProps = { movie }
  return pageProps
}
```

The `addContextProps()` hook always runs in Node.js,
which means SQL/ORM queries can be used to fetch data.

That's it, and we have actually already seen most of `vite-plugin-ssr`'s interface.

Thanks to the `render()` hook
you keep full control over how your pages are rendered,
and thanks to `*.page.client.js`,
you keep full control over the entire browser-side code.
This makes it *easy* and *natural* to use `vite-plugin-ssr` with any tool you want.

In short: `vite-plugin-ssr` is not only the most flexible, but also the easiest SSR tool out there.

<br/><br/>


## React Tour

Similarly to SSR frameworks,
pages are defined by page files.

```jsx
// /pages/index.page.jsx
// Environment: Browser, Node.js

import React, { useState } from "react";

export { Page };

function Page() {
  return <>
    This page is rendered to HTML and interactive: <Counter />
  </>;
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount((count) => count + 1)}>
      Counter {count}
    </button>
  );
}
```

By default, `vite-plugin-ssr` does filesystem routing:
```
FILESYSTEM                  URL
pages/index.page.jsx        /
pages/about.page.jsx        /about
```

You can also use Route Strings (for parameterized routes such as `/movies/:id`) and Route Functions (for full programmatic flexibility).

```js
// /pages/index.page.route.js
// Environment: Node.js

export default "/";
```

Unlike SSR frameworks,
*you* define how your pages are rendered.

```jsx
// /pages/_default.page.server.jsx
// Environment: Node.js

import ReactDOMServer from "react-dom/server";
import React from "react";
import { html } from "vite-plugin-ssr";

export { render };

async function render({ Page, pageProps }) {
  const viewHtml = ReactDOMServer.renderToString(
    <Page {...pageProps} />
  );

  return html`<!DOCTYPE html>
    <html>
      <head>
        <title>Vite w/ SSR Demo</title>
      </head>
      <body>
        <div id="page-view">${html.dangerouslySetHtml(viewHtml)}</div>
      </body>
    </html>`;
}
```
```jsx
// /pages/_default.page.client.jsx
// Environment: Browser

import ReactDOM from "react-dom";
import React from "react";
import { getPage } from "vite-plugin-ssr/client";

hydrate();

async function hydrate() {
  // (In production, the page is `<link rel="preload">`'d.)
  const { Page, pageProps } = await getPage();

  ReactDOM.hydrate(
    <Page {...pageProps} />,
    document.getElementById("page-view")
  );
}
```

The `render()` hook in `pages/_default.page.server.jsx` gives you full control over how your pages are rendered,
and `pages/_default.page.client.jsx` gives you full control over the browser-side code.
This control enables you to *easily* and *naturally*:
 - Use any tool you want such as React Router or Redux.
 - Use Preact, Inferno, Solid or any other React-like alternative.

Note how the files we created so far end with `.page.jsx`, `.page.route.js`, `.page.server.jsx`, and `.page.client.jsx`.
 - `.page.js`: defines the page's view that is rendered to HTML / the DOM.
 - `.page.client.js`: defines the page's browser-side code.
 - `.page.server.js`: defines the page's hooks (always run in Node.js).
 - `.page.route.js`: defines the page's Route String or Route function.

Using `vite-plugin-ssr` consists simply of writing these four types of files.

Instead of creating a `.page.client.js` and `.page.server.js` file for each page, you can create `_default.page.client.js` and `_default.page.server.js` which apply as default for all pages.

We already defined our `_default.*` files above,
which means that we can now create a new page simply by defining a new `.page.jsx` file.
(The `.page.route.js` file is optional and only needed if we want to define a parameterized route.)

The `_default.*` files can be overwriten. For example, you can create a page with a different browser-side code than your other pages.

```js
// /pages/about.page.client.js

// This file is empty which means that the `/about` page has zero browser-side JavaScript.
```
```jsx
// /pages/about.page.jsx

export { Page };

function Page() {
  return <>This page is only rendered to HTML.<>;
}
```

By overwriting `_default.page.server.js` you can
even render some of your pages with an entire different view framework such as Vue.

Note how files are collocated and share the same base `/pages/about.page.*`;
this is how you tell `vite-plugin-ssr` that `/pages/about.page.client.js` is the browser-side code of `/pages/about.page.jsx`.

Let's now have a look at how to fetch data for a page that has a parameterized route.

```jsx
// /pages/star-wars/movie.page.jsx
// Environment: Browser, Node.js

import React from "react";

export { Page };

function Page(pageProps) {
  const { movie } = pageProps;
  return <>
    <h1>{movie.title}</h1>
    <p>Release Date: {movie.release_date}</p>
    <p>Director: {movie.director}</p>
  </>;
}
```
```js
// /pages/star-wars/movie.page.route.js
// Environment: Node.js

export default "/star-wars/:movieId";
```
```js
// /pages/star-wars/movie.page.server.js
// Environment: Node.js

import fetch from "node-fetch";

export { addContextProps };
export { setPageProps };

async function addContextProps({ contextProps }) {
  // Route parameters are available at `contextProps`
  const { movieId } = contextProps;
  // We could also use SQL/ORM queries here
  const response = await fetch(`https://swapi.dev/api/films/${movieId}`);
  const movie = await response.json();
  return { movie };
}

// The `contextProps` are available only on the server, and only the `pageProps` are
// serialized and passed to the browser.
function setPageProps({ contextProps }) {
  // We select only the data we need in order to minimize what it sent over the network
  const { title, release_date, director } = contextProps.movie;
  const movie = { title, release_date, director };
  const pageProps = { movie };
  return pageProps;
}
```

The `addContextProps()` hook always runs in Node.js,
which means SQL/ORM queries can be used to fetch data.

That's it, and we have actually already seen most of `vite-plugin-ssr`'s interface.

Thanks to the `render()` hook
you keep full control over how your pages are rendered,
and thanks to `*.page.client.js`,
you keep full control over the entire browser-side code.
This makes it *easy* and *natural* to use `vite-plugin-ssr` with any tool you want.

In short: `vite-plugin-ssr` is not only the most flexible, but also the easiest SSR tool out there.

</details>

<br/><br/>


## Boilerplates

Scaffold a Vite app that uses `vite-plugin-ssr`.

With NPM:

```
npm init vite-plugin-ssr
```

With Yarn:

```
yarn create vite-plugin-ssr
```

Then choose between `vue`, `vue-ts`, `react`, and `react-ts`.

<br/><br/>


## Manual Installation

If you already have an existing Vite app:

1. Add `vite-plugin-ssr` to your `vite.config.js`.
   - [Vue](/create-vite-plugin-ssr/template-vue/vite.config.js)
   - [Vue + TypeScript](/create-vite-plugin-ssr/template-vue-ts/vite.config.ts)
   - [React](/create-vite-plugin-ssr/template-react/vite.config.js)
   - [React + TypeScript](/create-vite-plugin-ssr/template-react-ts/vite.config.ts)

2. Integrate `createRender()` with your server (Express.js, Koa, Hapi, Fastify, ...).
   - [Vue](/create-vite-plugin-ssr/template-vue/server/index.js)
   - [Vue + TypeScript](/create-vite-plugin-ssr/template-vue-ts/server/index.ts)
   - [React](/create-vite-plugin-ssr/template-react/server/index.js)
   - [React + TypeScript](/create-vite-plugin-ssr/template-react-ts/server/index.ts)

3. Define your `_default.page.client.js` and `_default.page.server.js`.
   - [Vue](/create-vite-plugin-ssr/template-vue/pages/_default/)
   - [Vue + TypeScript](/create-vite-plugin-ssr/template-vue-ts/pages/_default/)
   - [React](/create-vite-plugin-ssr/template-react/pages/_default/)
   - [React + TypeScript](/create-vite-plugin-ssr/template-react-ts/pages/_default/)

4. Create your first `index.page.js`.
   - [Vue](/create-vite-plugin-ssr/template-vue/pages/index/index.page.vue)
   - [Vue + TypeScript](/create-vite-plugin-ssr/template-vue-ts/pages/index/index.page.vue)
   - [React](/create-vite-plugin-ssr/template-react/pages/index/index.page.jsx)
   - [React + TypeScript](/create-vite-plugin-ssr/template-react-ts/pages/index/index.page.tsx)

5. Add the `dev` and `build` scripts to your `package.json`.
   - [Vue](/create-vite-plugin-ssr/template-vue/package.json)
   - [Vue + TypeScript](/create-vite-plugin-ssr/template-vue-ts/package.json)
   - [React](/create-vite-plugin-ssr/template-react/package.json)
   - [React + TypeScript](/create-vite-plugin-ssr/template-react-ts/package.json)

<br/><br/>


## Data Fetching

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

You fech data by using two hooks: `addContextProps()` and `setPageProps()`. The `async function addContextProps()` fetches data, while the `function setPageProps()` (not `async`) specifies what data is serialized and passed to the browser.

Hooks are called in Node.js, which means that you can use ORM/SQL database queries in your `addcontextprops()` hook.

Hooks are defined in `.page.server.js`.

```js
// /pages/movies.page.server.js
// Environment: Node.js

import fetch from "node-fetch";

export { addContextProps }
export { setPageProps }

async function addContextProps({ contextProps }) {
  const response = await fetch("https://api.imdb.com/api/movies/")
  const { movies } = await response.json()
  return { movies }
}

function setPageProps({ contextProps: { movies } }) {
  // We only select data we need: `vite-plugin-ssr` serializes and passes `pageProps`
  // to the client and we want to minimize what it sent over the network.
  movies = movies.map(({ title, release_date }) => ({title, release_date}))
  const pageProps = { movies }
  return pageProps
}
```

The `pageProps` are:
1. Passed to your `render()` hook.
2. Serialized and passed to the client-side.

```js
// /pages/_default.page.server.js
// Environment: Node.js

import { html } from 'vite-plugin-ssr'
import { renderToHtml, createElement } from 'some-view-framework'

export { render }

async function render({ Page, pageProps }) {
  // `Page` is defined below in `/pages/movies.page.js`.
  const pageHtml = await renderToHtml(createElement(Page, pageProps))
  /* Or if you use JSX:
  const pageHtml = await renderToHtml(<Page {...pageProps} />)
  */
  return html`<html>
    <div id='view-root'>
      ${html.dangerouslySetHtml(pageHtml)}
    </div>
  </html>`
}
```
```js
// /pages/_default.page.client.js
// Environment: Browser

import { getPage } from 'vite-plugin-ssr/client'
import { hydrateToDom, createElement } from 'some-view-framework'

hydrate()

async function hydrate() {
  const { Page, pageProps } = await getPage()
  await hydrateToDom(createElement(Page, pageProps), document.getElementById('view-root'))
  /* Or if you use JSX:
  await hydrateToDom(<Page {...pageProps} />, document.getElementById('view-root'))
  */
}
```
```js
// /pages/movies.page.js
// Environment: Browser, Node.js

export { Page }

function Page(pageProps) {
  const { movies } = pageProps
  /* ... */
}
```

<br/><br/>


## Routing

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

By default `vite-plugin-ssr` does Filesystem Routing.

```
FILESYSTEM                  URL
pages/index.page.js         /
pages/about.page.js         /about
pages/faq/index.page.js     /faq
```

For more control, you can define route strings in `.page.route.js` files.

```
// /pages/product.page.route.js

export default '/product/:productId'
```

The `productId` value is available at `contextProps.productId` so that you can fetch data in `async addContextProps({contextProps})` which is explained at [Data Fetching](#data-fetching).

For full programmatic flexibility, you can define route functions.

```js
// /pages/admin.page.route.js

// Route functions allow us to implement advanced routing such as route guards.
export default async ({ url, contextProps }) => {
  if (url==='/admin' && contextProps.user.isAdmin) {
    return { match: true }
  }
}
```

For detailed informations about Filesystem Routing, route strings, and route functions:
 - [API - Filesystem Routing](#filesystem-routing)
 - [API - `*.page.route.js`](#pageroutejs)

<br/><br/>


## Auth Data

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

Information about the authenticated user can be added to `contextProps` at the server integration point
[`createRender()`](#import--createrender--from-vite-plugin-ssr).
The `contextProps` are available to all hooks and route functions.

```js
const render = createRender(/*...*/)

app.get('*', async (req, res, next) => {
  const url = req.originalUrl
  // The `user` object, which holds user information, is provided by your
  // authentication middleware, for example the Express.js Passport middleware.
  const { user } = req
  const contextProps = { user }
  const html = await render({ url, contextProps })
  if (!html) return next()
  res.send(html)
})
```
<br/><br/>


## HTML `<head>`

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

HTML `<head>` tags such as `<title>` and `<meta>` are defined in the `render()` hook.

```js
// _default.page.server.js

import { html } from 'vite-plugin-ssr'
import { renderToHtml } from 'some-view-framework'

export { render }

async function render({ Page, contextProps }) {
  return html`<html>
    <head>
      <title>SpaceX</title>
      <meta name="description" content="We deliver your payload to space.">
    </head>
    <body>
      <div id="root">
        ${html.dangerouslySetHtml(await renderToHtml(Page))}
      </div>
    </body>
  </html>`
}
```

If you want to define `<title>` and `<meta>` tags on a page-by-page basis, you can use `contextProps`.

```js
// _default.page.server.js

import { html } from 'vite-plugin-ssr'
import { renderToHtml } from 'some-view-framework'

export { render }

async function render({ Page, contextProps }) {
  // We use `contextProps.docHtml` which pages can define with their `addContextProps()` hook
  let title = contextProps.docHtml.title
  let description = contextProps.docHtml.description

  // Defaults
  title = title || 'SpaceX'
  description = description || 'We deliver your payload to space.'

  return html`<html>
    <head>
      <title>${title}</title>
      <meta name="description" content="${description}">
    </head>
    <body>
      <div id="root">
        ${html.dangerouslySetHtml(await renderToHtml(Page))}
      </div>
    </body>
  </html>`
}
```
```js
// about.page.server.js

export { addContextProps }

function addContextProps() {
  const docHtml = {
    // This title and description will overwrite the defaults
    title: 'About SpaceX',
    description: 'Our mission is to explore the galaxy.'
  }
  return { docHtml }
}
```

If you want to define `<head>` tags by some deeply nested view component, you can use libraries such as [@vueuse/head](https://github.com/vueuse/head) or [react-helmet](https://github.com/nfl/react-helmet).
(Thanks to `vite-plugin-ssr`'s design you can easily use and integrate any tool you want, including these two libraries.)
We recommend using such libraries only if you have a strong rationale:
the aforementioned solution using `contextProps` is simpler and works in the vast majority of cases.

<br/><br/>


## Store

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

Even complex integrations, such as Vuex or Redux, are simple and straightforward to implement.
Because you control how your pages are rendered,
integration is just a matter of following the official guide of the tool you want to integrate.

While you can follow the official guides *exactly* as they are (including serializing and injecting the initial state into HTML),
you can also leverage `vite-plugin-ssr`'s `pageProps` to make your life slightly easier,
as shown in the following examples.

 - [/examples/vuex/](/examples/vuex/)
 - [/examples/redux/](/examples/redux/)

<br/><br/>


## Markdown

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

You can use `vite-plugin-ssr` with any Vite markdown plugin.

For Vue you can use `vite-plugin-md`:
 - [/examples/vue/pages/markdown.page.md](/examples/vue/pages/markdown.page.md)
 - [/examples/vue/vite.config.ts](/examples/vue/vite.config.ts)

For React you can use `@brillout/vite-plugin-mdx`:
 - [/examples/react/pages/markdown.page.md](/examples/react/pages/markdown.page.md)
 - [/examples/react/vite.config.ts](/examples/react/vite.config.ts)

<br/><br/>


## Pre-rendering

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

> :warning: Pre-rendering is currenlty being worked on. If you want an ETA, open a GitHub issue.

> :asterisk: **What is pre-rendering?**
> Pre-rendering means to render the HTML of all your pages at once.
> Normally, the HTML of a page is rendered at request-time
> (when your user goes to your website).
> With pre-rendering, the HTML of a page is rendered at build-time instead
> (when yun run `vite-plugin-ssr prerender`).
> Your app then consists only of static assets (HTML, JS, CSS, images, ...)
> and you can deploy your app to so-called "static hosts" such as [GitHub Pages](https://pages.github.com/) or [Netlify](https://www.netlify.com/).
> Without pre-rendering, you need to use a Node.js server that renders your pages' HTML at request-time.

To pre-render your pages, run `npx vite && npx vite --ssr && npx vite-plugin-ssr prerender`. (Or with Yarn: `yarn vite && yarn vite --ssr && yarn vite-plugin-ssr prerender`.)

For pages with a parameterized route (e.g. `/movie/:movieId`), you'll have to use the [`prerender()` hook](#export--prerender-).

The `prerender()` hook can also be used to prefetch data for multiple pages at once.

<br/><br/>


## Page Redirection

> :warning: We recommend reading the [Vue Tour](#vue-tour) or [React Tour](#react-tour) before proceeding with guides.

Your `render()` hook doesn't have to return HTML and can, for example, return `{ redirectTo: '/some/url' }` in order to do a URL redirect.

```js
export { render }

function render({ contextProps }) {
  // If the user goes to `/movie/42` but there is no movie with ID `42` then
  // we redirect the user to `/movie/add` so he can add a new movie.
  if (contextProps.movieId === null) {
    return { redirectTo: '/movie/add' }
  } else {
    // The usual render stuff
    // ...
  }
}
```
```js
const render = createRender(/*...*/)

app.get('*', async (req, res, next) => {
  const url = req.originalUrl
  const contextProps = {}
  const renderResult = await render({ url, contextProps })
  if (renderResult?.redirectTo) {
    res.redirect(307, '/movie/add')
  } else if (typeof renderResult === 'string') {
    res.send(renderResult)
  } else {
    next()
  }
})
```

<br/><br/>


## `*.page.js`

Environment: `Browser`, `Node.js`
<br>
[Ext Glob](https://github.com/micromatch/micromatch#extglobs): `/**/*.page.*([a-zA-Z0-9])`

A `*.page.js` file should have a `export { Page }`. (Or a `export default`.)

`Page` represents the page's view that is rendered to HTML / the DOM.

`vite-plugin-ssr` doesn't do anything with `Page` and just passes it untouched to:
 - Your `render({ Page })` hook.
 - The client-side.

```js
// *.page.js
// Environment: Browser, Node.js

export { Page }

// We export a JSX component, but we could as well export a Vue/Svelte/... component,
// or even export some totally custom object since vite-plugin-ssr doesn't do anything
// with `Page`: it just passes it to your `render()` hook and to the client-side.
function Page() {
  return <>Hello</>
}
```

```js
// *.page.server.js
// Environment: Node.js

import { html } from 'vite-plugin-ssr'
import renderToHtml from 'some-view-framework'

export { render }

// `Page` is passed to the `render()` hook
async function render({ Page }) {
  const pageHtml = await renderToHtml(Page)

  return html`<html>
    <body>
      <div id="root">
        ${html.dangerouslySetHtml(pageHtml)}
      </div>
    </body>
  </html>`
}
```
```js
// *.page.client.js
// Environment: Browser

import { getPage } from 'vite-plugin-ssr/client'
import { hydrateToDom } from 'some-view-framework'

hydrate()

async function hydrate() {
  // `Page` is available in the browser.
  const { Page } = await getPage()
  await hydrateToDom(Page)
}
```

The `*.page.js` file is lazy-loaded only when needed, that is when an HTTP request matches the page's route.

<br/><br/>


## `*.page.client.js`

Environment: `Browser`
<br>
[Ext Glob](https://github.com/micromatch/micromatch#extglobs): `/**/*.page.client.*([a-zA-Z0-9])`

A `.page.client.js` file is a `.page.js`-adjacent file that defines the page's browser-side code.

It represents the *entire* browser-side code. This means that if you create an empty `.page.client.js` file, then the page has zero browser-side JavaScript.
(Except of Vite's dev code when not in production.)

This also means that you have full control over the browser-side code: not only can you render/hydrate your pages as you wish, but you can also easily integrate browser libraries.

```js
// *.page.client.js

import { getPage } from 'vite-plugin-ssr/client'
import { hydrateToDom, createElement } from 'some-view-framework'
import GoogleAnalytics from '@brillout/google-analytics'

main()

async function main() {
  analytics_init()
  analytics.event('[hydration] begin')
  await hydrate()
  analytics.event('[hydration] end')
}

async function hydrate() {
  const { Page, pageProps } = await getPage()
  await hydrateToDom(createElement(Page, pageProps), document.getElementById('view-root'))
}

let analytics
function analytics_init() {
  analytics = new GoogleAnalytics('UA-121991291')
}
```

<br/>

### `import { getPage } from 'vite-plugin-ssr/client'`

Environment: `Browser`

The `async getPage()` function provides `Page` and `pageProps` for the browser-side code `.page.client.js`.

```js
// /pages/demo.page.client.js

import { getPage } from 'vite-plugin-ssr/client'

hydrate()

async function hydrate() {
  const { Page, pageProps } = await getPage()
  /* ... */
}
```

- `Page` is the `export { Page }` (or `export default`) of the `/pages/demo.page.js` file.
- `pageProps` is the value returned by your `setPageProps()` function (which you define and export in the adjacent `pages/demo.page.server.js` file).

The `pageProps` are serialized and passed from the server to the browser with [`devalue`](https://github.com/Rich-Harris/devalue).

In development `getPage()` dynamically `import()` the page, while in production the page is preloaded (with `<link rel="preload">`).

<br/><br/>


## `*.page.route.js`

Environment: `Node.js`
<br>
[Ext Glob](https://github.com/micromatch/micromatch#extglobs): `/**/*.page.route.*([a-zA-Z0-9])`

The `*.page.route.js` files enable further control over routing with:
 - Route Strings
 - Route Functions

<br/>

### Route String

For a page `/pages/film.page.js`, a route string can be defined in a `/pages/film.page.route.js` adjacent file.

```js
// /pages/film.page.route.js

// Match URLs `/film/1`, `/film/2`, ...
export default '/film/:filmId'
```

If the URL matches, the value of `filmId` is available at `contextProps.filmId`.

The syntax of route strings is based on [`path-to-regexp`](https://github.com/pillarjs/path-to-regexp)
(the most widespread route syntax in JavaScript).
For user friendlier docs, check out the [Express.js Routing Docs](https://expressjs.com/en/guide/routing.html#route-parameters)
(Express.js uses `path-to-regexp`).

<br/>

### Route Function

Route functions give you full programmatic flexibility to define your routing logic.

```js
// /pages/film/admin.page.route.js

export default async ({ url, contextProps }) {
  // Route functions allow us to implement advanced routing such as route guards.
  if (! contextProps.user.isAdmin) {
    return {match: false}
  }
  // We can use RegExp and any JavaScript tool we want.
  if (! /\/film\/[0-9]+\/admin/.test(url)) {
    return {match: false}
  }
  filmId = url.split('/')[2]
  return {
    match: true,
    // Add `filmId` to `contextProps`
    contextProps: { filmId }
  }
}
```

The `match` value can be a (negative) number which enables you to resolve route conflicts.
The higher the number, the higher the priority.
For example, `vite-plugin-ssr` internally defines `_404.page.js`'s route as:

```js
// node_modules/vite-plugin-ssr/.../_404.page.route.js

// Ensure lowest priority for the 404 page
export default () => ({match: -Infinity})
```

<br/><br/>


## `*.page.server.js`

Environment: `Node.js`
<br>
[Ext Glob](https://github.com/micromatch/micromatch#extglobs): `/**/*.page.server.*([a-zA-Z0-9])`

A `.page.server.js` file is a `.page.js`-adjacent file that exports the page's hooks:
- `export { addContextProps }`
- `export { setPageProps }`
- `export { render }`
- `export { prerender }`

The `*.page.server.js` file is lazy-loaded only when needed.

<br/>

### `export { addContextProps }`

The `addContextProps()` hook is used to provide further `contextProps` values.

The `contextProps` are passed to all hooks (which are defined in `.page.server.js`) and to the route function (if there is one defined in `.page.route.js`).

You can provide initial `contextProps` values at your server integration point: [`const render = createRender(/*...*/); render({ url, contextProps })`](#import--createrender--from-vite-plugin-ssr).
Which you usually use to pass information about the authenticated user,
see [Auth Data](#auth-data) guide.

The `addContextProps()` hook is usually used in conjunction with the [`setPageProps()` hook](#export--setpageprops-) to fetch data, see [Data Fetching](#data-fetching) guide.

Since `addContextProps()` is always called in Node.js, ORM/SQL database queries can be used.

```js
// /pages/movies.page.server.js

import fetch from "node-fetch";

export { addContextProps }

async function addContextProps({ contextProps, Page }){
  const response = await fetch("https://api.imdb.com/api/movies/")
  const { movies } = await response.json()
  /* Or with an ORM:
  const movies = Movie.findAll() */
  /* Or with SQL:
  const movies = sql`SELECT * FROM movies;` */
  return { movies }
}
```

- `Page` is the `export { Page }` (or `export default`) of the `.page.js` file.
- `contextProps` is the initial accumulation of:
   1. The `contextProps` you provided in your the server integration point `createRender()`.
   2. The route parameters (such as `contextProps.movieId` for a page with a route string `/movie/:movieId`).

<br/>

### `export { setPageProps }`

The `setPageProps()` hook provides the `pageProps` which are consumed by `Page`.

The `pageProps` are serialized and passed from the server to the browser with [`devalue`](https://github.com/Rich-Harris/devalue).

It is usally used in conjunction with the `addContextProps()` hook: data is fetched in `addContextProps()` and then made available to `Page` with `setPageProps()`.

```js
// /pages/movies.page.server.js
// Environment: Node.js

import fetch from "node-fetch";

async function addContextProps({ contextProps }) {
  const response = await fetch("https://api.imdb.com/api/movies/")
  const { movies } = await response.json()
  return { movies }
}

function setPageProps({ contextProps: { movies } }) {
  // We remove data we don't need: `vite-plugin-ssr` serializes and passes `pageProps`
  // to the client and we want to minimize what it sent over the network.
  movies = movies.map(({ title, release_date }) => ({title, release_date}))
  const pageProps = { movies }
  return pageProps
}
```
```js
// /pages/movies.page.js
// Environment: Browser, Node.js

export { Page }

function Page(pageProps) {
  const { movies } = pageProps
  /* ... */
}
```

<br/>

### `export { prerender }`

> :asterisk: Check out the [Pre-rendering Guide](#pre-rendering) to get an overview about pre-rendering.

The `prerender()` hook enables parameterized routes (e.g. `/movie/:movieId`) to be pre-rendered:
by defining the `prerender()` hook you provide the list of URLs (`/movie/1`, `/movie/2`, ...) and (optionally) the `contextProps` of each URL.

If you don't have any parameterized route,
then you can prerender your app without defining any `prerender()` hook.
You can, however, still use the `prerender()` hook
to increase the effeciency of pre-rendering as
it enables you to fetch data for multiple pages at once.

```js
// /pages/movie.page.route.js

export default '/movie/:movieId`
```
```js
// /pages/movie.page.server.js

export { prerender }

async function prerender() {
  const movies = await Movie.findAll()

  const moviePages = (
    movies
    .map(movie => {
      const url = `/movie/${movie.id}`
      const contextProps = { movie }
      return {
        url,
        // Beacuse we already provide the `contextProps`, vite-plugin-ssr will *not* call
        // the `addContextProps()` hook.
        contextProps
      }
      // We could also return `url` wtihout `contextProps`. In that case vite-plugin-ssr would
      // call `addContextProps()`. But that would be wasteful since we already have all the data
      // of all movies from our `await Movie.findAll()` call.
      // return { url }
    })
  )

  // We can also return URLs that don't match the page's route.
  // That way we can provide the `contextProps` of other pages.
  // Here we provide the `contextProps` of the `/movies` page since
  // we already have the data.
  const movieListPage = {
    url: '/movies', // The `/movies` URL doesn't belong to the page's route `/movie/:movieId`
    contextProps: {
      movieList: movies.map(({id, title}) => ({id, title})
    }
  }

  return [movieListPage, ...moviePages]
}
```

The `prerender()` hook is only used when pre-rendering:
if you don't call
`vite-plugin-ssr prerender`
then no `prerender()` hook is called.

<br/>

### `export { render }`

The `render()` hook renders `Page` to an HTML string.

Note that the `render()` hook can also return something else than HTML,
for example an object `{ redirectTo: '/some/url' }` in order to do [Page Redirection](#page-redirection).

```js
// *.page.server.js

import { html } from 'vite-plugin-ssr'
import { renderToHtml, createElement } from 'some-view-framework'

export { render }

async function render({ Page, pageProps, contextProps }){
  const pageHtml = await renderToHtml(createElement(Page, pageProps))

  const title = contextProps.title || 'My SSR App'

  return html`<!DOCTYPE html>
    <html>
      <head>
        <title>${title}</title>
      </head>
      <body>
        <div id="page-root">${html.dangerouslySetHtml(pageHtml)}</div>
      </body>
    </html>`
}
```

- `Page` is the `export { Page }` (or `export default`) of the `.page.js` file.
- `pageProps` is the value returned by the `setPageProps()` hook.
- `contextProps` is the accumulation of:
   1. The `contextProps` you passed to [`const render = createRender(/*...*/); render({ url, contextProps })`](#import--createrender--from-vite-plugin-ssr).
   2. The route parameters (such as `contextProps.movieId` for a page with a route string `/movie/:movieId`).
   3. The `contextProps` you returned in your `addContextProps()` hook (if you defined one).

<br/>

### `import { html } from 'vite-plugin-ssr'`

Environment: `Node.js`

The `html` tag sanitizes HTML (to prevent XSS injections).
It is usually used in your `render()` hook defined in `.page.server.js`.

```js
// *.page.server.js

import { html } from 'vite-plugin-ssr'

export { render }

async function render() {
  const title = 'Hello<script src="https://devil.org/evil-code"></script>'
  const pageHtml = "<div>I'm already <b>sanitized</b>, e.g. by Vue/React</div>"

  // We're safe because `html` sanitizes `title`
  return html`<!DOCTYPE html>
    <html>
      <head>
        <title>${title}</title>
      </head>
      <body>
        <div id="page-root">${html.dangerouslySetHtml(pageHtml)}</div>
      </body>
    </html>`
}
```

All strings, e.g. `title`, are automatically sanitized (technically speaking: HTML-escaped)
so that you can safely include untrusted strings
such as user-generated text.

The `html.dangerouslySetHtml(str)` function injects the string `str` as-is *without* sanitizing.
It should be used with caution and
only for HTML strings that are guaranteed to be already sanitized.
It is usually used to include HTML generated by React/Vue/Solid/... as these frameworks always generate sanitized HTML.
If you find yourself using `html.dangerouslySetHtml()` in other situations be extra careful as you run into the risk of creating a security breach.

You can assemble the overall HTML document from several pieces of HTML segments.
For example, when you want some HTML parts to be included only for certain pages:

```js
// _default.page.server.js

import { html } from 'vite-plugin-ssr'
import { renderToHtml } from 'some-view-framework'

export { render }

async function render({ Page, contextProps }) {
  // We only include the `<meta name="description">` tag if the page has a description.
  // (A page can define `contextProps.docHtml.description` with its `addContextProps()` hook.)
  const description = contextProps.docHtml?.description
  let descriptionTag = ''
  if( description ) {
    // Note how we use the `html` tag for an HTML segment
    descriptionTag = html`<meta name="description" content="${description}">`
  }

  // We use the `html` tag again for the overall HTML
  // `descriptionTag` is already sanitized and will not be sanitized again
  return html`<html>
    <head>
      ${descriptionTag}
    </head>
    <body>
      <div id="root">
        ${html.dangerouslySetHtml(await renderToHtml(Page))}
      </div>
    </body>
  </html>`
}
```

<br/><br/>


## `_default.*`

The `_default.page.server.js` and `_default.page.client.js` files are like regular `.page.server.js` and `.page.client.js` files, but they are special in the sense that they don't apply to a single page file; instead, they apply as a default to all pages. 

There can be several `_default.*` files.

```
marketing/_default.page.server.js
marketing/_default.page.client.js
marketing/index.page.js
marketing/about.page.js
marketing/jobs.page.js
admin-panel/_default.page.server.js
admin-panel/_default.page.client.js
admin-panel/index.page.js
```

The `marketing/_default.*` files apply to the `marketing/*.page.js` files, while
the `admin-panel/_default.*` files apply to the `admin-panel/*.page.js` files.

The `_default.page.server.js` and `_default.page.client.js` files are not adjacent to any `.page.js` file, and
defining `_default.page.js` or `_default.page.route.js` is forbidden.

<br/><br/>


## `_404.page.js`

The `_404.page.js` page is like any other page with the exception that it has a predefined route.

```js
// node_modules/vite-plugin-ssr/.../_404.page.route.js

// Ensure lowest priority for the 404 page
export default () => ({match: -Infinity})
```

<br/><br/>


## `_500.page.js`

The `_500.page.js` page is shown to your user when an error occurs.

It is optional. (If there is no `_500.page.js`, then `_404.page.js` is shown instead.)

You define it like any other page. (You create a `_500.page.js`, and you can define `_500.page.client.js` and `_500.page.server.js`.)

<br/><br/>


## Filesystem Routing

By default a page is mapped to a URL based on where its `.page.js` file is located.

```
FILESYSTEM                        URL              COMMENT
pages/about.page.js               /about
pages/index/index.page.js         /                (`index` is mapped to the empty string)
pages/HELLO.page.js               /hello           (Mapping is done lower case)
```

The `pages/` directory is optional and you can save your `.page.js` files wherever you want.

```
FILESYSTEM                        URL
user/list.page.js                 /user/list
user/create.page.js               /user/create
todo/list.page.js                 /todo/list
todo/create.page.js               /todo/create
```

The directory common to all your `*.page.js` files is considered the routing root.

For more control over routing, define route strings or route functions in [`*.page.route.js`](#pageroutejs).

<br/><br/>


## `import { createRender } from 'vite-plugin-ssr'`

Environment: `Node.js`

The `createRender()` is the integration point between your server and `vite-plugin-ssr`.

```js
const render = createRender({ viteDevServer, isProduction, root })

app.get('*', async (req, res, next) => {
  const url = req.originalUrl
  const contextProps = {}
  const html = await render({ url, contextProps })
  if (!html) return next()
  res.send(html)
})
```

- `isProduction` is a boolean. When set to `true`, `vite-plugin-ssr` loads already-transpiled code from `dist/` instead of on-the-fly transpiling code.
- `root` is a string holding the absolute path of your app's root directory. All your `.page.js` files should be a descendent of the root directory.
- `viteDevServer` is the value returned by `const viteDevServer = await vite.createServer(/*...*/)`.

Since `render({ url, contextProps})` is agnostic to Express.js, you can use `vite-plugin-ssr` with any server framework such as Koa, Hapi, Fastify, or vanilla Node.js.

Examples:
 - [JavaScript](/create-vite-plugin-ssr/template-react/server/index.js)
 - [TypeScript](/create-vite-plugin-ssr/template-react-ts/server/index.ts)

<br/><br/>


## `import vitePlugin from 'vite-plugin-ssr'`

Environment: `Node.js`

The Vite plugin has no options.

```js
// vite.config.js

const ssr = require("vite-plugin-ssr");

module.exports = {
  plugins: [ssr()]
};
```

<br/><br/>


