---
title: React Props are arguments passed to components
publish: true
---

- props stands for properties
    
- props: in HTML format, we need to write them as attribute,
    
    => then it will be passed to components as a object from function or other components
    
    => so we need to destruct the obj when component accept theme
    
    ```js
      const myElement = <Car brand="Ford" />;
    
      function Car(props) {
      return <h2>I am a { props.brand }!</h2>;
    }
    ```
    
- React props are read-only you will get and error when you change them



