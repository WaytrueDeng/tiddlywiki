---
title: how to get a new string list contains certain character
publish: true
---
## the problem is 
this is problem met by me when I scan a directory, get the files name, but I want to get only markdown format. so I write following code

```js
const fileNames = fs.readdirSync(postsDirectory);
  const regex = new RegExp('\.md$') //①
  const mdfileNames = fileNames.map(
  (filename) => {
    if (regex.test(filename))  {
    return filename
    } //②

  }

```
- ①:I decide to use regular express's test method, you may have notice that the "new" key word, we have use it in [[how_do_we_assign_obejct_to_class_in_JS]], so I guess
  
  ②: I try to write a if statement but I forget how to write else statement so I have to learn [[how_to_write_if_statement_in_JS]],so we change the js like below
	```js
    const fileNames = fs.readdirSync(postsDirectory);
  const regex = new RegExp('\.md$')
  const mdfileNames = fileNames.map(
  (filename) => {
    if (regex.test(filename)) {
    return filename;
    } else {
    return;
    }
  }
  )

	```
     => we find a new problem that the value  dose not match the regex,  returned as defined, then also are read by file sync function so we may not return anything just break; this method also failed until I filnd a new method 
## the solution

use the list.filter method 
```js
  const fileNames = fs.readdirSync(postsDirectory);
  const regex = new RegExp('\.md$')
  /*const mdfileNames = fileNames.map(
  (filename) => {
    if (regex.test(filename)) {
    return filename;
    } 
  }
  )*/

  const mdfileNames =fileNames.filter(item => regex.test(item));

```