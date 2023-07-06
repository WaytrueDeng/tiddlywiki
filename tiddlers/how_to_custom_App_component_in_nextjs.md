---
title: how to custom App component in nextjs
publish: true
---

Next.js uses the `App` component to initialize pages. You can override it and control the page initialization and:

- Persist layouts between page changes
- Keeping state when navigating pages
- Inject additional data into pages
- [Add global CSS](https://nextjs.org/docs/pages/building-your-application/styling)

``` js
export default function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```

- The `Component` prop is the active `page`, so whenever you navigate between routes, `Component` will change to the new `page`. Therefore, any props you send to `Component` will be received by the `page`.
- `pageProps` is an object with the initial props that were preloaded for your page by one of our [data fetching methods](https://nextjs.org/docs/pages/building-your-application/data-fetching), otherwise it's an empty object.