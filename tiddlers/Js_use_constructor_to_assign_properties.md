---
title: Js use constructor to assign properties
publish: true
---

constructor function is inside the class function  [[JS_class_is_a_function]], it accept the parameter of then assign it to the class . so it’s also a method, a special one

the constructor function is called automatically when we initialize a object of Class. so  [[how_do_we_assign_obejct_to_class_in_JS]]


```js
  class className {
  constructor (para){
  this.propName = para;
  }
}
```



you can find the interesting thing that  [[class_function_no_parameters]]

and you may have noticed [[that_this_keyword_refers_to_current_object]]