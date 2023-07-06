---
title: JSX combines HTML and JS
publish: true
---

- you may ask what is JSX?
    
    > JSX stands for JavaScript XML.
    > 
    > JSX allows us to write HTML in React.
    > 
    > JSX makes it easier to write and add HTML in React.
    
- JSX auto convert HTML tag into React element so you don’t have to mannual create it
    
    ```js
    const myElement = <h1>I Love JSX!</h1>;
    
    const myElement = React.createElement('h1', {}, 'I do not use JSX!');
    ```
    
- JSX will be translated in to JS in runtime
    
- you can write JS expression in JSX’s HTML tag
    
    => you need to put the expression inside the curly braces “{}” then it will be executed
    
    ```js
    const myElement = <h1>React is {5 + 5} times better with JSX</h1>;
    ```
    
- when you write multiline HTML you need to put them into a parentheses()
    
    ```js
      const myElement = (
      <ul>
        <li>Apples</li>
        <li>Bananas</li>
        <li>Cherries</li>
      </ul>
    );
    ```
    
- JS component must have one top level, if you have multi element, you have to wrap them in to one parent element
    
    => to prevent adding extra node(because you have to add parent node), you can use fragment
    
    ```jsx
      const myElement = (
      <>
        <p>I am a paragraph.</p>
        <p>I am a paragraph too.</p>
      </>
    );
    ```
    
- JSX HTML tag must be closed, JS following XML rule, single tag must be closed to
    
    ```js
    const myElement = <input type="text" />;
    ```
    
- HTML class attribute conflicts with JS keyword so we change it to className
    
    ```jsx
    const myElement = <h1 className="myclass">Hello World</h1>;
    ```
    
- you can’t use if statement inside JSX, so you need to outside it or if you want to use it inside you should use [[ternary_operator_is_short_for_if_statement]]



