---
title: Link component client-side navigation
publish: true
---

when we use traditional a tag in html pointing to other page, the browser will refresh and send new request, it will be slow.

in React we use Link component which pre-fetch the data and use javascript to change the content in the page, do not need the  browser refresh

```jsx
import Link from 'next/link';

export default function FirstPost() {
  return (
    <>
      <h1>First Post</h1>
      <h2>
        <Link href="/">Back to home</Link>
      </h2>
    </>
  );
}
```



