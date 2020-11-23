---
title: Internationalization with Sapper and Svelte
author: anand
date: 2020-09-28T08:11:06.102Z
---

As a Swiss company, it’s extremely important to support at least two languages in our web app: English and German. If possible, we should also support other languages like French. So, we decided to build our website with internationalization best-practices in mind.

## Requirements

### URL structure

We selected the URL structure that works best for the web, with language and ISO code, for example https://example.com/en-us for English (United States). So, if someone wants to visit the About page, they’d go to https://example.com/en-us/about, and we’ll also redirect https://example.com/about based on their preferred language.

## What didn’t work

### Custom build process

The problem is that Sapper doesn’t have any i18n built-in, so we had to reinvent the wheel. We decided that the best approach to generate these URLs is to do just that — use Sapper’s file structure to generate pages in each language directory. However, keeping pages in sync is hard, so here’s the strategy we applied:

1. We renamed the components and routes folders to `_components` and `_routes` respectively
1. We do all the work in these new directories, and add the original ones to `.gitignore`
1. In Svelte files, we use `|hello|` which is replaced with the translation
1. Our build script auto-generates the “real” components and routes in each language; for example, `components/en-us/...`

### Why this didn’t work

1. The build process was using fs.watch and that means running an extra process during compilation
1. Pages took quite a while to be generated (more than 50ms) so Sapper would try to re-build the site multiple times on each change

## What worked

1. We switched back to the default `components` and `routes` structure that Sapper recommends
1. We created the `en-us` subdirectory under routes and created all pages there
1. We create an i18n helper instead of replacing translations directly in Svelte files
1. We copy the `en-us` directory to other languages within routes as part of the build proces

##Implementation
In our build process, we copy the en-ch directory to other languages and let Sapper handle the routing:

```js
import { copy } from "fs-extra";
import { join } from "path";

export const generateSvelte = async () => {
  for await (const language of ["de-ch", "fr-ch"]) {
    await copy(
      join(".", "src", "routes", "en-ch"),
      join(".", "src", "routes", language)
    );
  }
};
```

When we want to add a translation, we use the `t` helper:

```html
<script>
  import { stores } from "@sapper/app";
  import { t } from "../helpers/i18n";
  const { page } = stores();

  // The URL is something like /en-ch/hello, so language is "en-ch"
  $: language = $page.path.split("/")[1];
</script>

<h1>{t(language, 'hello')}</h1>
```

This is the source of the translation helper. It just tries to find the right translation from a huge object:

```ts
/**
 * Get the translated string for a key in a language
 * @param {string} language - Language, e.g., en-ch
 * @param {string} key - Translation key
 */
export const t = (language, key) => {
  if (
    typeof languages[language] === "object" &&
    typeof languages[language][key] === "string"
  )
    return languages[language][key];
  return languages["en-ch"][key] ?? "";
};
```

In the above helper function, languages is an object with a structure like so:

```json
{
  "en-ch": {
    "hello": "Hello"
  },
  "de-ch": {
    "hello": "Hallo"
  }
}
```

We supply a string rather than an object and use `JSON.parse()` for better performance. In the future, we’re planning on adding lazy loading of translations (above and beyond Svelte’s treeshaking) and continue working on better internationalization.
