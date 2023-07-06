---
title: how to add static file to nextjs
publish: true
---

you always need to import some static file like css, custom css, and images

- file in public folder can be accessed from root of the  application

## add images to your web site 
- we use \<img> tag in tradition, but we have to face some problem 
	- we have to manual optimize the image size to save band width or something
	- you have to manual set the different picture size to fit different device
	- you have to manual assure that the picture loaded after the view port really be visited 

## add third party to you web 
- we used to use script tag to add js, but you  can't control when to load it and it is a rendering process which will impact the web performance
   => nextjs provide Script component to use 
```jsx
<Script
        src="https://connect.facebook.net/en_US/sdk.js"
        strategy="lazyOnload"
        onLoad={() =>
          console.log(`script loaded correctly, window.FB has been populated`)
        }
      />
```
## add css to your web

we need to create a component to wrap our css, so you have to know[[how_to_create_your_own_component]] ,beacuse [[nextjs_css_can_be_module_specific]]

and if you want to global css you need look at [[how_to_add_global_css_in_nextjs]]
