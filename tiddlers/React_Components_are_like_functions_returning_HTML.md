---
title: React Components are like functions returning HTML
publish: true
---

there are two types of React Component: class one and function one, nowadays we focus on the function one

- function component is much less code than class one
    
    ```js
    function Car() {
    return <h2>Hi, I am a Car!</h2>;
    }
    ```
    
- you can use <foo /> format to render a component
    
    ```js
    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<Car />);
    ```

- [[divide_component_into_file_for_better_reuse]]

