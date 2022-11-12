---
title: "Release: @vueuse/head v1 🎉"
description: "Learn about the new @vueuse/head v1 release including new features and breaking changes"
publishedAt: '2022-11-12'
excerpt: 'Learn about the new @vueuse/head v1 release including new features and breaking changes.'
status: published
image: "https://harlanzw.com/social/how-vite-works.png"
tags: 
  - vue
---

Hey there, firstly welcome to my blog. My name is Harlan and I took over the maintenance of [@vueuse/head](https://github.com/vueuse/head) from [EGOIST](https://github.com/sponsors/egoist)
in October.

EGOIST did an amazing job with the package, and I'm grateful to continue the work he started.

I'll be providing some background information for this release, and then go over the new features and breaking changes. 
Feel free to jump straight to the [release notes](#v100-release).

## Changes leading to up to v1

My goals for taking of maintenance was closing all open issues for the package, improve performance, documentation and DX.

Notable changes:
- Full `useHead` TypeScript support powered by [zhead](https://github.com/harlan-zw/zhead) - [6f919c7](https://github.com/vueuse/head/pull/85)
- Computer getter support - [b6d74dbeb](https://github.com/vueuse/head/commit/b6d74dbebd32f772923f722c399091c73e21b6ed)
- Significant performance improvements - [e1bc8d2](https://github.com/vueuse/head/commit/e1bc8d2e35a9104adbd3a62c29bdadb89181b3fc), [691bcc8](https://github.com/vueuse/head/commit/691bcc88ae3526879238995866daeec39e8bc4c6)
- Vue 2.7 support - [d6ece59c3d843a5fa82d7c2bb29e76b1c73ac05d](https://github.com/vueuse/head/pull/147) (thanks to vetruvet)

## Why a major bump - v1?

While most issues were closed and easy to solve, there was a major outstanding issue that I wanted to address, *Server Only Tags* (see [discussion](https://github.com/nuxt/framework/discussions/7785)). 
There were also issues were around tags disappearing unexpectedly when hydrating.

Both of these were blocked by how the DOM patching was designed. The strategy was to add state to the DOM and use this
to determine what tags to remove.

```html
<html data-head-attrs="lang,dir">
<head>
<meta name="head:count" content="29">
</head>
<body data-head-attrs="class">
</body>
</html>
```

You can see in the above, we're tracking the attributes being added and the number of tags. The `head:count` element
is used as an anchor to start rendering tags from.

To make the issues clear:
- DOM updates are aggressive, they override whatever value is in the spot regardless if it's a valid tag from `@vueuse/head`
- They remove any tags which end up in the `head:count` anchors scope
- State in DOM isn't so nice

To solve these issues, a new DOM patching algorithm was needed that tracked side effects gracefully.

## Unhead

This was a major piece of work, and I figured if I'm going to have to refactor the entire package, I should aim to also solve
another outstanding issue, _Universal Head Management_.

So I started on [Unhead](https://github.com/harlan-zw/unhead), a Universal document <head> tag manager, that's tiny, adaptable, and full-featured.

And I'm happy to say that it's now being used in @vueuse/head v1.

Unhead has first-party support for Vue, but there is planned work to support many other frameworks (React, Svelte, etc).

With this in mind @vueuse/head is a thin wrapper for unhead, but provides legacy head functions (i.e `addHeadObjs`) and
provides the `<Head>` component.

## v1.0.0 Release

### 🤖 Core

Now powered by [Unhead](https://github.com/harlan-zw/unhead).

Featuring a new DOM patching algorithm that tracks side effects gracefully, less aggressive removal of tags and attributes

### ✨ Enhancements

- Vue 2.7 Support
- Options API Support
- New docs [unhead.harlanzw.com](https://unhead.harlanzw.com/) (WIP)
- Tag deduping is now vastly improved. It's likely you won't need `key` anymore. See [tag deduping](https://unhead.harlanzw.com/guide/guides/handling-duplicates)

### 🚀 New Features

#### useServerHead

[Documentation](https://unhead.harlanzw.com/guide/guides/server-only-tags)

Lets you render tags on the server only. This has the same API as `useHead`.

```ts
useServerHead({
  scripts: [
    {
      // this wouldn't work on the client, so we use useServerHead
      src: () => import('~/assets/my-script.js?url'),
    }
  ]
})
```

#### useTagMetaFlat

Define meta tags in a flat object, fully typed.

```ts
useTagMetaFlat({
  description: 'My about page',
  ogDescription: 'Still about my about page',
  ogTitle: 'About',
  ogImage: 'https://example.com/image.png',
  twitterCard: 'summary_large_image',
})
```

#### tagPosition

[Documentation](https://unhead.harlanzw.com/guide/guides/positions)

Lets you define the position of a tag in the DOM.

```ts
useHead({
  script: [
    {
      src: 'https://example.com/script.js',
      tagPosition: 'bodyOpen',
    }
  ]
})
```

#### tagPriority

[Documentation](https://unhead.harlanzw.com/guide/guides/sorting)

Lets you define the priority of a tag with a number or string.

```ts
useHead({
  script: [{ key: 'not-important', src: '/not-important-script.js',},],
})
useHead({
  script: [
    {
      // script is the tag name to target, `not-important` is the key we're targeting  
      tagPriority: 'before:script:not-important',
      src: '/must-be-first-script.js',
    },
  ],
})
```


#### tagDuplicateStrategy

[Documentation](https://unhead.harlanzw.com/guide/guides/handling-duplicates#tagduplicatestrategy)



#### Meta content array support

[Documentation](https://unhead.harlanzw.com/guide/guides/handling-duplicates#using-arrays-with-meta)

Reduce boilerplate by using arrays for meta tags.

```ts
useHead({
  meta: [
    {
      name: 'og:image',
      content: [
        'https://example.com/image.png',
        'https://example.com/image2.png',
      ],
    },
  ],
})
```


#### DOM Event Handlers

[Documentation](https://unhead.harlanzw.com/guide/guides/dom-event-handling)

Function support for DOM event handlers.

```ts
useHead({
  bodyAttrs: {
    onresize: (e) => {
      console.log('resized', e)
    }
  },
  script: [
    {
      src: 'https://example.com/analytics.js',
      onload: (el) => {
        console.log('loaded', el)
      }
    }
  ]
})
```

#### Hooks

Engine is now powered by hooks, provided by [hookable](https://github.com/unjs/hookable). This allows you to hook into
any of the core functionality.

See [API hooks](https://unhead.harlanzw.com/api/core/create-head) and [Infer SEO MetaTags](https://unhead.harlanzw.com/guide/recipes/infer-seo-meta-tags), not documented properly yet.

#### New shortcut composables

Same API as `useHead`, but targeted as a specific tag type.

- useTagTitle
- useTagBase
- useTagMeta
- useTagLink
- useTagScript
- useTagStyle
- useTagNoscript
- useHtmlAttrs
- useBodyAttrs
- useTitleTemplate

## Migration Guide

⚠️ Breaking changes are minimal, but there are some changes to be aware of.

Please report any issues you find and they will fixed be promptly.

You may consider using [@unhead/vue](https://unhead.harlanzw.com/integrations/vue/setup) directly if you [don't need @vueuse/head](https://unhead.harlanzw.com/integrations/vue/vueuse-head).

### Verify your tags

The new DOM patching algorithm has not been tested in all possible scenarios, it's possible that there are unforeseen edge cases.  

### `htmlAttrs` and `bodyAttrs` merge strategy

If you had built your code around with the assumption that setting `htmlAttrs` or `bodyAttrs` would clear the old tags, this is now different.

```ts
// old
useHead({
  htmlAttrs: {
    class: 'my-class',
  },
})

useHead({
  htmlAttrs: {
    class: 'new-class',
  },
})

// <html class="new-class">
```

```ts
// new
useHead({
  htmlAttrs: {
    class: 'my-class',
  },
})

useHead({
  htmlAttrs: {
    class: 'new-class',
  },
})

// <html class="my-class new-class">
```

Check the [documentation](https://unhead.harlanzw.com/guide/guides/handling-duplicates#tagduplicatestrategy) to learn more.


### Duplicate tags in the same entry allowed

To make duplicate tag handling more intuitive, duplicate tags are now allowed in the same entry.

Having these metas across `useHead`'s will still dedupe as before.

```ts
// old
useHead({
  meta: [
    { name: 'description', content: 'foo' },
    { name: 'description', content: 'bar' },
  ],
})

// only the last tag is rendered
// <meta name="description" content="bar">
```

```ts
// old
useHead({
  meta: [
    { name: 'description', content: 'foo' },
    { name: 'description', content: 'bar' },
  ],
})

// both are rendered
// <meta name="description" content="foo">
// <meta name="description" content="bar">
```

## Next Steps

I am aiming to have this in Nuxt v3 stable soon. If you have any ideas on the future of unhead and @vueuse/head 
please get in touch with me through an issue, Discord or Twitter.

Thanks for reading!