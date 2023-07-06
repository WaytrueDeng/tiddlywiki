---
title: ternary operator is short for if statement
publish: true
---

```js
if (authenticated) {
  renderApp();
} else {
  renderLogin();
}

authenticated ? renderApp() : renderLogin();
```



