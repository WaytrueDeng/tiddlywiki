---
title: React goal is to render HTML
publish: true
---

- React renders HTML to the web page by using a function called createRoot() and its method render().
    
    => the createRoot() function takes one argument, an HTML element.The purpose of the function is to define the HTML element where a React component should be displayed.
    
    => The render() method is then called to define the React component that should be rendered.
    
    ```js
    const container = document.getElementById('root');
    const root = ReactDOM.createRoot(container);
    root.render(<p>Hello</p>);
    ```
    
    => you may ask have what is the element named by “root”,we never define it how could React get it?, it’s not the \<html> tag, it’s defined in the pulic folder in your create-react-app index.html
- [[not_need_to_be_root_when_use_createRoot]]


