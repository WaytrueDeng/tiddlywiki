---
title: divide component into file for better reuse
publish: true
---

you can write component in a .js file then you export it and import it

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import Car from './Car.js';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<Car />);
```



