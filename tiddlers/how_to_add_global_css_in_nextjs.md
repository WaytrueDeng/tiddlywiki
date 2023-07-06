---
title: how to add global css in nextjs
publish: true
---


you need to create a file named \_app.js under pages folder, you have noticed that it is not a component so you do not place it under component directory

```jsx 
export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

\_app.js is a top-level React component that wraps all the pages in your application. You can use this component to keep state when navigating between pages, if you want to know more about \_app.js you need to see [[how to custom App component in nextjs]]