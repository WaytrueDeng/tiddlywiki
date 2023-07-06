---
title: order is important in destrutcting
publish: true
---

destructing is the process of assgin multi value from array or object to multi variable

```js


const vehicles = ['mustang', 'f-150', 'expedition'];

const [car,, suv] = vehicles;

const vehicleOne = {
  brand: 'Ford',
  model: 'Mustang',
  type: 'car',
  year: 2021,
  color: 'red',
  registration: {
    city: 'Houston',
    state: 'Texas',
    country: 'USA'
  }
}

myVehicle(vehicleOne)

function myVehicle({ model, registration: { state } }) {
  const message = 'My ' + model + ' is registered in ' + state + '.';
}
```

- we can ommit some value
- we can notice that



