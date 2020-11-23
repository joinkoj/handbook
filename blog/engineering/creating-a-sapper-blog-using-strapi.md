---
title: Creating a Sapper blog using Strapi
author: anand
date: 2020-10-01T11:12:06.102Z
---

We just published our first few blog posts, and after exploring several different content management systems, we decided to use [Strapi](https://strapi.io/). Strapi is an open-source headless CMS written in Node.js; it works with most database management systems and it’s super easy to deploy.

In this quick guide, we’ll go through our process of using the Strapi API with Svelte and Sapper to generate a blog with categories and author pages.

## Prerequisites

Before heading to the Svelte code, you should create the following content types in Strapi:

1. Blog categories
2. Blog posts
3. Team members

Each content type contains a `title` and `slug`, and blog posts and team members also include a rich text field named `content`.

## Code

We’ll use `fs-extra` for writing files, `axios` to make requests, `markdown-it` to render markdown, and `highlight.js` for syntax highlighting. Get started by installing these dependencies:

```bash
npm install axios markdown-it highlight-js fs-extra
```

If you use TypeScript like we do, you’ll need these types installed as well:

```bash
npm install --save-dev @types/axios @types/fs-extra @types/highlight.js @types/markdown-it
```

Then, we create a directory `blog` in `src/routes` and create a JSON API route `[slug].json.js`:

```js
import { readFile } from "fs-extra";
import { join } from "path";

export const get = async (req, res) => {
  const { slug } = req.params;
  let file = "";
  try {
    file = await readFile(
      join(".", "src", "routes", "blog", "data", `${slug}.json`),
      "utf8"
    );
  } catch (error) {}
  if (file) {
    res.writeHead(200, {
      "Content-Type": "application/json",
    });
    return res.end(file);
  } else {
    res.writeHead(404, {
      "Content-Type": "application/json",
    });
    return res.end(
      JSON.stringify({
        message: "Not found",
      })
    );
  }
};
```

This endpoint will return the contents of blog posts by locating them as JSON files in the `data` directory. We will programmatically write these files using a helper. Let’s create a file in the project root, `strapi.ts`:

```ts
import axios from "axios";
import { ensureFile, writeJson } from "fs-extra";
import { join } from "path";
import Markdown from "markdown-it";
import highlight from "highlight.js";

const markdown = new Markdown({
  typographer: true,
  linkify: true,
  highlight: (str, lang) => {
    try {
      return highlight.highlight(lang, str).value;
    } catch (error) {}
    return "";
  },
});

const contentTypes = [
  { path: "blog/data", api: "blog-posts" },
  { path: "blog/categories/data", api: "blog-categories" },
  { path: "blog/authors/data", api: "team-members" },
];

export const strapi = async () => {
  for await (const type of contentTypes) {
    console.log("Strapi: Fetching", type);
    let { data } = await axios.get(`https://your-api-url.com/${type.api}`);
    if (Array.isArray(data)) {
      data = data.filter((i) => {
        if (typeof i.draft === "boolean") return !i.draft;
        if (typeof i.published === "boolean") return i.published;
        return true;
      });
    }
    for await (const item of data) {
      const slug = item.slug ?? item.id;
      if (typeof item.content === "string") item.content = markdown.render(item.content);
      Object.keys(item).forEach((key) => {
        if (Array.isArray(item[key]))
          item[key] = item[key].map((author: any) => {
            if (typeof author.content === "string")
              author.content = markdown.render(author.content);
            return author;
          });
      });
      await ensureFile(join(".", "src", "routes", type.path, `${slug}.json`));
      await writeJson(join(".", "src", "routes", type.path, `${slug}.json`), item);
    }
    await ensureFile(join(".", "src", "routes", type.path, "all.json"));
    await writeJson(join(".", "src", "routes", "type.path, "all.json"), data);
  }
};
strapi();
```

With the above helper, we generate the following files:

1. `src/routes/blog/data/all.json` contains all the blog posts
1. `src/routes/blog/data/SLUG.json` contains each blog post with its slug
1. `src/routes/blog/authors/data/all.json` contains all the authors
1. `src/routes/blog/authors/data/SLUG.json` contains each author
1. `src/routes/blog/categories/data/all.json` contains all the authors
1. `src/routes/blog/categories/data/SLUG.json` contains a single author

To generate these, run this script following:

```bash
npx ts-node strapi.ts
```

Then, let’s create some routes. The `index.svelte` file in `src/routes/blog` will list all blog posts:

```html
<script context="module">
  export async function preload() {
    const res = await this.fetch("blog/all.json");
    const data = await res.json();
    if (res.status === 200) {
      return { posts: data };
    } else {
      this.error(res.status, data.message);
    }
  }
</script>

<script>
  export let posts;
</script>

{#each posts as post}
  <article>
    <header>
      <h2><a href={`/blog/${post.slug}/`}>{post.title}</a></h2>
      <div class="meta">
        {#each post.team_members as author}
          <a href={`/blog/authors/${author.slug}/`}>
            <img alt="" src={author.profile_picture} />
            <div>{author.title}</div>
          </a>
        {/each}
        <div class="d-flex">
          {#each post.blog_categories as category}
            <a href={`/blog/categories/${category.slug}/`}>
              <span>{category.title}</span>
            </a>
          {/each}
          <a href={`/blog/${post.slug}/`}>
            <span>{new Date(post.created_at).toLocaleString('en-US', {
                year: 'numeric',
                month: 'long',
                day: 'numeric',
              })}</span>
          </a>
        </div>
      </div>
    </header>
    <div>
      {@html (post.content ?? '').split('\n')[0]}
    </div>
    <footer class="buttons">
      <a
        class="button"
        href={`/blog/${post.slug}/`}>{t(language, 'blog.continueReading')}
        &rarr;</a>
    </footer>
  </article>
{/each}
```

Then, we create the single blog post page `[slug.svelte]`:

```html
<script context="module">
  export async function preload({ params }) {
    const res = await this.fetch(`blog/${params.slug}.json`);
    const data = await res.json();
    if (res.status === 200) {
      return { post: data };
    } else {
      this.error(res.status, data.message);
    }
  }
</script>

<script>
  export let post;
</script>

<svelte:head>
  <title>{post.title}</title>
</svelte:head>

<h1>{post.title}</h1>

<div class="meta d-flex">
  <a href={`/${language}/blog/${post.slug}/`}>
    <span>{new Date(post.created_at).toLocaleString('en-US', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      })}</span>
  </a>
  {#each post.blog_categories as category}
    <a href={`/${language}/blog/categories/${category.slug}/`}> <span>{category.title}</span> </a>
  {/each}
</div>

<div class="blog-post">
  {@html post.content}
</div>

<footer>
  {#each post.team_members as author}
    <div class="row">
      <div class="col-md-3">
        <a href={`/blog/authors/${author.slug}/`}>
          <img alt="" src={author.profile_picture} />
        </a>
      </div>
      <div class="col-md-9 pl-0">
        <h2>
          <div>{t(language, 'blog.aboutAuthor')}</div>
          <div>{author.title}</div>
        </h2>
        <div>
          {@html (author.content ?? '').split('\n')[0]}
        </div>
        <nav>
          <a href={`/blog/authors/${author.slug}/`}>
            {t(language, 'blog.aboutAuthorMore').replace('$NAME', (author.title ?? '').split(' ')[0])}
          </a>
        </nav>
      </div>
    </div>
  {/each}
</footer>
```

Lastly, we repeat the process and create the `index.svelte` and `[slug].svelte` routes in the categories and authors directories under `blog`, replacing the API URLs.
