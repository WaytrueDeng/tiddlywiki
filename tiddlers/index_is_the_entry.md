---
title: index is the entry
publish: true
---
the index.js under the page folder will be the root setgment of your url eg: http://127.0.0.1:3000/
```sh
❯ exa -aT pages
pages
└── index.js
```

the index.js under pages export a default function called Home, it returns a div which contain some components, you cant see i
``` js
export default function Home() 

/*...*/
return (
<div>
<Head>
</Head>
<Main>
</Main>
</div>
)
```


