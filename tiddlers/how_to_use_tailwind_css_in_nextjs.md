---
title: how to use tailwind css in nextjs
publish: true
---


```shell
npm install -D tailwindcss autoprefixer postcss
```

- [[how_to_solve_gradient_color_background_duplicated]]

as for other stuff you could refer to their website doc or nextjs doc, it's easy to install 

but you have to notion that we can set the default vaue for the page use the @layer base 

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  html {
    @apply  bg-fixed bg-gradient-to-t from-slate-950/90 to-slate-800/90 text-slate-200 ;
  }
  body {
    min-height: 100%

  }
  h1 {
    @apply text-sky-200 text-4xl text-center;
  }
  h2 {
    @apply text-xl;
  }
  pre {
    max-height: 300px;
    @apply overflow-auto;
  }
  a {
    @apply text-blue-400 hover:bg-slate-200 underline underline-offset-1 decoration-2;
  }

  p {
    @apply pt-2;
  }

  /* width */
::-webkit-scrollbar {
  width: 10px;
}

/* Track */
::-webkit-scrollbar-track {
  background: rgb(30 41 59);
}
 
/* Handle */
::-webkit-scrollbar-thumb {
  background:  rgb(51 65 85);; 
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: rgb(6 182 212);; 
}

  /* ... */
}

```
