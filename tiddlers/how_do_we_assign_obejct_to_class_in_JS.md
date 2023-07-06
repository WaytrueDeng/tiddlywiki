---
title: how do we assign obejct to class in JS
publish: true
---

we know that  [[JS_class_is_a_function]], so we must to call it, it’s also special because we need to use the “new” keyword to assign it

```js
class Car {
  constructor(name) {
    this.brand = name;
  }

  present() {
    return 'I have a ' + this.brand;
  }
}

const mycar = new Car("Ford");
mycar.present();
```

you can notice that we can pass parameters to the car function, but we say that  [[class_function_no_parameters]]

## Review
this page's information is useful when I try to solve [[how_to_get_a_new_string_list_contains_certain_character]]

