---
title: how to create your own component
publish: true
---

let's craete a component director in you top-level directory
``` sh
exa -DTL 1
.
├── components
├── node_modules
├── pages
├── public
└── styles
```

we write out Layout component named layout.js
```jsx
export default function Layout({ children }) {
  return <div>{children}</div>;
}
```
=> you may ask what the children is ? if we use \<Layout>this is Children\</Layout>
     the thing wrapped in the open and close tag is it's children, will be passed to the component as name of children in a obejct, so you can destruct it. you may want to look [[order_is_important_in_destrutcting]]

you can then import the Layout component in your post
```jsx
import Head from 'next/head';
import Link from 'next/link';
import Layout from '../../components/layout'; ①

export default function FirstPost() {
  return (
    <Layout>
      <Head>
        <title>First Post</title>
      </Head>
      <h1>First Post</h1>
      <h2>
        <Link href="/">← Back to home</Link>
      </h2>
    </Layout>
  );
}
```

- you need to notice that ①: the relative path is used to load the componentz

=> now Layout component has wrapped all item on this page


