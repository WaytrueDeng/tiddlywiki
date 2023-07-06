---
title: spread operator to quickly copy
publish: true
---

> The JavaScript spread operator (…) allows us to quickly copy all or part of an existing array or object into another array or object.

often used in combination of destructuring

```js
const numbers = [1, 2, 3, 4, 5, 6];

const [one, two, ...rest] = numbers;
```

spread operator can also used in object

```js
const myVehicle = {
  brand: 'Ford',
  model: 'Mustang',
  color: 'red'
}

const updateMyVehicle = {
  type: 'car',
  year: 2021,
  color: 'yellow'
}

const myUpdatedVehicle = {...myVehicle, ...updateMyVehicle}
```

you need to know the difference of {myVehicle,updateMyVehicle} and the above …



