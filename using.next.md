# next.js
This tutorial is simplified / adapted from the official [Next.js tutorial](https://nextjs.org/learn/basics/create-nextjs-app), which you should definitely take a look at if you're interested in using Next.js.

## Getting Started
First, initialize your project by running:
`npx create-next-app@latest`

- `cd` into the new folder that you created, and run `npm run dev`. This will start the Next.js development
server. Go to `http://localhost:3000` to see the page running.

- Try making some simple edits to `pages/index.js` to see the the page automatically updated. 

- Go ahead and get rid of everything inside of the `<main>` tag. Put a `<h1>` tag inside to create a header.

- OK, let's add a new page. Make a copy of `index.js` and rename it `about.js`. Change the page to have different information in it from the original file. Once you save the file, you should be able to go to `http://localhost:3000/about` and see the page running.

- Let's link these together. Go to your `index.js` page and add the following include at the top of the file:
`import Link from 'next/link'`

- Add a link to the `<main>` tag on your `index.js` and save your file:
`<Link href='./about'>About</Link>`

If you open your index page now you should see the link appear. Go ahead and do the same thing on your about page but link back to your index page.
`<Link href='./'>Home</Link>`

OK, here's where things get strange (in a cool way). If you go into the console and change the styles (try setting `background:red` on the `<body>` tag to make things obvious) and then click on a link, you'll notice that the style is persistent. What happened?

When you use the `<Link>` tag, data is prefetched for linked resources. When you click on the link, instead of fetching a new file Next.js dynamically inserts the pre-fetched / pre-rendered html in to the current page. You'll notice that, because of this, switching between pages is instantaneous... there is no "blanking" that occurs. Try switching the tags back to regular `<a>` and notice the effect this has on speed.

Lots of Next.js React components that seem unncessary actually do really cool things under the hood (like `Link`). For example, the `Image` component will automatically scale your image to the correct size based on the current viewport. It will also wait to download images until the optimal time, so that page loads can be fast as possible. Finally, it will convert your images to optimized compressed formats (like `WebP`) without requiring any additional actions on your part, making downloads as small as possible.

## Make a navigation component
1. Place the following file in your `pages` directory:

```js
import styles from '../styles/Home.module.css'
import Link from 'next/link'

export default function Nav() {
  return (
    <>
    <ul>
      <li><Link href='./'>Home</Link></li>
      <li><Link href='./about'>about</Link></li>
      <li><Link href='./blog'>blog</Link></li>
    </ul>
    </>
  )
}
```

2. In your `about.js` and `index.js` import the navigation: `import Nav from './nav.js'`
3. You can now add a `<Nav />` element to each of your pages.

## Using a server function
There are lots of different strategies for rendering pages in Next.js. For static pages, we can render them on the server without much difficulty. We'll do a quick example of reading through a list of files in a folder (imagine a directory of blog entires) and then printing them to the screen.

1. Create a new folder called `posts` and a new folder called `lib` in the top level of your project.
2. Place two very simple, single-sentence text files into your new `posts` directory.
3. We'll use our `lib` directory as a place to store our server functions for rendering static files. Create a file in there named `posts.js` and put in the following code:

```js
import fs from 'fs'
import path from 'path'

const postsDirectory = path.join( process.cwd(), 'posts' )

export function getPosts() {
  const fileNames = fs.readdirSync( postsDirectory )
  
  const allPostsData = fileNames.map( fileName => {
    const id = fileName.replace(/\.txt$/, '')
    const fullPath = path.join( postsDirectory, fileName )
    const text = fs.readFileSync( fullPath, 'utf8' )

    return { id,tex }
  })

  return allPostsData
}
```

In the above code, we read through all the files with `.txt` in it, get the text for each file, and then return an object containing the filename and the associated text.

4. In our `pages` directory, create a new file called `blog.js` and change it to the follwing:

```js
import { getPosts } from '../lib/posts'
import Nav from './nav.js'

export function getStaticProps() {
  const posts = getPosts()
  return { props: { posts } }
}

export default function Home({ posts })  {
  return (
    <>
      <h1>Blog</h1>
      <Nav />
      {
        posts.map( post => (<h3 key={post.id}>{post.text}</h3>) )
      }
    </>
  )
}
```

`getStaticProps` is a magic keyword to Next.js that we're going to use static web content to create our page. We call the function we wrote on the server, and the results our used to populate the `props` of our `Home` component. Then we loop through the props and create a `h3` element for each list item we find.
