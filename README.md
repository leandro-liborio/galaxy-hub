# The New Galaxy Community Hub

This is a prototype of the replacement for the current community hub at [galaxyproject.org](https://galaxyproject.org). To experience it right now, visit [preview.galaxyproject.org](https://preview.galaxyproject.org).

For a summary of the why and how, our poster from GCC2021 might help:

![Poster summary of the new hub](doc/poster.svg)

## Getting started

First, make sure you have [Node](https://nodejs.org/en/) installed. Then, you'll need a package manager. These instructions use [yarn](https://yarnpkg.com/), but there are equivalent commands for [npm](https://docs.npmjs.com/cli/v7/commands/npm).

You can get the site running locally by first cloning this repo:  
\- The `-b gridhub` tells it to check out the `gridhub` branch, where this prototype lives for now.
```sh
$ git clone -b gridhub 'https://github.com/galaxyproject/galaxy-hub.git'
```

Then install the dependencies:
```sh
$ cd galaxy-hub
$ yarn install
```

Then you can build the site in development mode to run it in a local server (at http://localhost:8080) and see your content:
```sh
$ yarn develop
```
This command includes a hot reloader which will update the site automatically each time you edit a file.  
Note: There's a [known issue](https://github.com/galaxyproject/galaxy-hub/issues/748) that can occur when you're renaming/moving/deleting directories or image files. If you're finding the development server crashes too often, try adding the arguments `--resource copy` to the `yarn develop` command. This fixes the issue, at the cost of a far larger build directory (because all images are copied to it instead of linked).

To generate the static files for the entire site, just run `build` instead:
```sh
$ yarn build
```
The static files can then be found in the `dist` directory.

## Creating static pages

### File organization

To see an example of how the files are organized, see the [`src`](/NickSto/galaxy-hub/tree/gridhub/src) directory of the [`gridhub`](/NickSto/galaxy-hub/tree/gridhub) branch of [`galaxy-hub`](/galaxyproject/galaxy-hub).

For static pages (normal, informational pages), you create a directory, whose name becomes the last part of the url. Then you create an `index.md` file inside it. The url will be everything *after* `content` and *before* `index.md`:

| Path to Markdown file                        | URL path                      |
|:---------------------------------------------|:----------------------------- |
| `content/events/2021-02-gtn/index.md`        | `/events/2021-02-gtn/`        |
| `content/galaxy-project/statistics/index.md` | `/galaxy-project/statistics/` |

### Writing the Markdown

See the [Authoring Guide](doc/AUTHORING.md)

### Custom layouts for static pages

By default, any `index.md` you create pretty much anywhere in `content/` displayed the same way. It's got a good generic layout for most pages you'd want to create. But sometimes you'd like a special layout for a certain class of pages, like entries in a directory of people. This framework comes with a good example: the Galaxy Platform Directory. Any `index.md` created under `content/use/` will be displayed like [this](https://galaxyproject.org/use/globus-genomics/).

You can do this with a custom [Collection](https://gridsome.org/docs/collections/). These are defined in the `collections` setting in `config.json`:
```json
  "collections": {
    "Platform": "/use/"
  },
```
Just add an entry like the existing one. The key is the name of your Collection and the value is the url path you want it to appear at. This will also be the path in the `content/` directory your `index.md` files will live at. **Note**: each `index.md` file must be in a directory exactly one level below the given directory. So `content/use/anvil/index.md` will be a `Platform`, but `content/use/archive/deeptools/index.md` will not.

Then you can create the custom layout for the pages in this collection. This is done by creating a `.vue` file in the `src/templates/` directory. The file should have the same name as your collection. E.g. the template for the `Platform` collection is `src/templates/Platform.vue`. FYI, if you'd like to modify the default layout, it's defined in `src/templates/Article.vue`. Every `index.md` that's *not* in one of the directories defined in the `collections` in `config.json` is part of the `Article` Collection.

### Categorizing your static pages

Pages in the `Article` collection can be subgrouped into different categories. These let you easily query for a specific category when listing `Article`s on a dynamic page. Pages are assigned a category based on their location in the filesystem (and thus their url).

The `categories` setting in `config.json` defines each category of pages by its url path:
```json
  "categories": {
    "/blog":  "blog",
    "/events": "events",
    "/news": "news",
    "/careers": "careers"
  },
```
In this example, all urls one level below `/events/` will be put in the `events` category. **Note** that the url must be exactly one level below, not deeper! So `/events/gcc2019/` will match, but not `/events/gcc2019/abstracts/`.

To define a new category, just add another entry, with the parent url as the key (**without** an ending slash) and the category name as the value. Then you can query for pages in that category in your `.vue` file in the `src/pages/` directory.

## Creating dynamic pages

Dynamic pages are ones where part of it is auto-generated. Usually they list a certain group of static pages.

### File organization

These are created by making a `.vue` file in the `src/pages/` directory. Much like the Markdown files, the placement of the `.vue` file determines the url. So `src/pages/Blog.vue` displays at `/blog/` and `src/pages/news/Gtn.vue` would display at `/news/gtn/`. There's also the `src/mediated-pages/` directory, which works the same except pages there have access to some custom variables set by the framework.

### Querying for the auto-generated content

The "dynamic" bit of dynamic pages is almost always a list of other pages. For example, `/news/` lists all the news articles and `/events/webinars/` lists all the pages on webinars.

To get this list of pages, you use the `<page-query>` part of your `.vue` file. This is a [GraphQL](https://graphql.org/) query enhanced with [custom filters](https://gridsome.org/docs/filtering-data/) provided by Gridsome. The Gridsome docs provide more detail on `<page-query>`s [here](https://gridsome.org/docs/querying-data/).

To learn how to query for pages you need to understand how they are organized the framework. There are two, hierarchical types of page groupings. The top level is the [Collection](https://gridsome.org/docs/collections/). Collections are described in more detail [above](#custom-layouts-for-static-pages). Basically, every page belongs to the `Article` collection unless it's part of a specialized one you define (like `Platform` or `Person`). Each collection has its own way of displaying a page. Then, `Article`s are divided into categories like `blog`, `events`, or `careers`. More detail on categories [above](#categorizing-your-static-pages).

To query for all pages of a given collection, you prepend the collection name with `all` in the query:
```graphql
  allPlatform {
    totalCount
    edges {
      node {
        id
        title
        path
      }
    }
  }
```
To query for pages in a specific category, you add a filter for it. This query retrieves all `Article`s in the `events` category:
```graphql
  allArticle(filter: {category: {eq: "events"}}) {
    totalCount
    edges {
      node {
        id
        title
        path
      }
    }
  }
```

### Writing static content for dynamic pages

Dynamic pages show auto-generated content, but they also usually have some expository static text or images above or below the dynamic content. Instead of keeping this content in the `.vue` framework files, you can keep it in its natural form: Markdown files.

Just make a file named `main.md` in the directory that corresponds to the url of your dynamic page. Then in the `.vue` file, you can query for it and then insert it into your `<template>` with a line like:
```vue
<div v-html="$page.main.content" />
```
To make it available to your template, you also need to add a clause to your `<page-query>` like this:
```graphql
  main: insert (path: "/insert:events/main/") {
    id
    title
    content
  }
```
..where `events/` is the url of the dynamic page and `main` is the base name of the Markdown file. You can actually name your Markdown file anything as long as you replace that part of the query. A common one is `footer.md` for content that goes below the dynamic content.

FYI, under the hood this is done with a custom [Collection](https://gridsome.org/docs/collections/) named `Insert`. Any `.md` file *not* named `index.md` automatically becomes an `Insert`.
