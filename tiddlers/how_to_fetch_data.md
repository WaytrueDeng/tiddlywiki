---
title: how to fetch data
publish: true
---

when you export a page as component, you can also export a async function called getStaticProps. (you have to know that [`getStaticProps`](https://nextjs.org/docs/basic-features/data-fetching#getstaticprops-static-generation) can only be exported from a [**page**](https://nextjs.org/docs/basic-features/pages). You can’t export it from non-page files.)
- it will run at build time.
- you can send the data as props to the page
  ```jsx
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```


we use markdown file as the posts data, on our disk .

create a folder named post to store our markdown file, markdown file has yaml frontmatter, we need `gray-matter` to parse them.
```shell
npm install gray-matter
```

we also need a function to read files, so we create top-level directory named lib(unlink page folder, you can name lib or something else) and create the post.js file

```js
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

const postsDirectory = path.join(process.cwd() /*①*/, 'posts');

export function getSortedPostsData() {
  // Get file names under /posts
  const fileNames = fs.readdirSync(postsDirectory); /*②*/
  const allPostsData = fileNames.map((fileName) => {
    // Remove ".md" from file name to get id
    const id = fileName.replace(/\.md$/, '');

    // Read markdown file as string
    const fullPath = path.join(postsDirectory, fileName);
    const fileContents = fs.readFileSync(fullPath, 'utf8');

    // Use gray-matter to parse the post metadata section
    const matterResult = matter(fileContents);

    // Combine the data with the id
    return {
      id,
      ...matterResult.data,
    };
  });
  // Sort posts by date
  return allPostsData.sort((a, b) => {
    if (a.date < b.date) {
      return 1;
    } else {
      return -1;
    }
  });
}
```
- ①: to get the posts directory in which we put markdown file
   ```js
   > process.cwd()
'/Users/waytrue/Documents/workround/nextjs-blog'
```
   ②: you will get all files name in the directory
   ``` js
      console.log(fs.readdirSync(postsDirectory))
  '.git',
  '.obsidian',
  '.zk',
  'DOM_is_a_programming_interface.md',
  'DOM_just_like_the_abstruct_structue_of_html.md',
  'ES_is_the_blueprint_of_the_JS.md',
  'How_does_React_work.md',
  'JSX_combines_HTML_and_JS.md',
  'JS_class_is_a_function.md',
```

this is failed because my directory contain subdirectory, which can be read by the fileSync, so I have to filter them out.        
=> [[how_to_get_a_new_string_list_contains_certain_character]]?
