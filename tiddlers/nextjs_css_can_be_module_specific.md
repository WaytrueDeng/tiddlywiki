---
title: nextjs css can be module specific
publish: true
---

nextjs give us the power to use css on specific module, in other words, css just works on certain component, this is because the nextjs will add random prefix or off-fix to  your class name, your class is finally unique

- we create a css file under components folder
```sh
nextjs-blog/components on  main [!?] via  v18.14.0
❯ exa -T
.
├── layout.js
└── layout.module.css
```

```css
❯ bat layout.module.css
───────┬───────────────────────────────────────────────────────────
       │ File: layout.module.css
───────┼───────────────────────────────────────────────────────────
   1   │ .container {
   2   │   max-width: 36rem;
   3   │   padding: 0 1rem;
   4   │   margin: 3rem auto 6rem;
   5   │ }
   6   │
───────┴──────────────────────────────────────────────────────────
```

you have to know that module css file must be name by the `.module.css` form

you will change your layout.js to 
```jsx
import styles from './layout.module.css';

export default function Layout({ children }) {
  return <div className={styles.container}>{children}</div>; <=① 
}
```
you may have notice that ①class attribute has changed to className you can find why on [[JSX_combines_HTML_and_JS]]


at last, sometime you may want to css work on every page, you need to know[[how_to_add_global_css_in_nextjs]]