# ramblings

A tiny Jekyll blog served by GitHub Pages at <https://erndip.github.io/ramblings/>.

## Writing a post

Add a file to `_posts/` named `YYYY-MM-DD-slug.md`:

```md
---
title: My post title
---

Markdown goes here.
```

Push it. GitHub Pages rebuilds in about a minute.

- The date comes from the filename. The URL is `/ramblings/slug/`.
- The front page always shows the newest post, and every page lists all posts.
- **Don't rename a published post.** Renaming changes its URL, which breaks links and detaches its comment thread.

## URLs

| What | URL |
|---|---|
| Latest post + archive | `https://erndip.github.io/ramblings/` |
| A single post | `https://erndip.github.io/ramblings/<slug>/` |
| Atom feed | `https://erndip.github.io/ramblings/feed.xml` |

## One-time setup

1. The repo must be **public** (giscus needs this).
2. Go to Settings → Pages → Source: *Deploy from a branch* → `main` / `(root)`.
3. Go to Settings → General → Features and enable **Discussions**.
4. Install the giscus app on this repo: <https://github.com/apps/giscus>
5. Go to <https://giscus.app>, enter `erndip/ramblings`, and pick the **Announcements** category. Copy `data-repo-id` and `data-category-id` into the `giscus:` block in `_config.yml`.

## Showing the latest post on erndip.github.io

Drop this component into the parent site's `src/components/` and render `<LatestRambling />` wherever you want it.
GitHub Pages serves the feed with `Access-Control-Allow-Origin: *`, so the absolute URL also works from `npm run dev`.

```tsx
import { useEffect, useState } from 'react'

type Post = { title: string; url: string; date: string; summary: string }

const FEED = 'https://erndip.github.io/ramblings/feed.xml'

const text = (el: Element | null) => el?.textContent?.trim() ?? ''
const stripHtml = (html: string) =>
  new DOMParser().parseFromString(html, 'text/html').body.textContent?.trim() ?? ''

export default function LatestRambling() {
  const [post, setPost] = useState<Post | null>(null)

  useEffect(() => {
    fetch(FEED)
      .then(r => r.text())
      .then(xml => {
        const entry = new DOMParser().parseFromString(xml, 'application/xml').querySelector('entry')
        if (!entry) return
        setPost({
          title: text(entry.querySelector('title')),
          url: entry.querySelector('link')?.getAttribute('href') ?? '',
          date: new Date(text(entry.querySelector('published'))).toLocaleDateString(undefined, { dateStyle: 'long' }),
          summary: stripHtml(text(entry.querySelector('summary'))),
        })
      })
      .catch(() => {})
  }, [])

  if (!post) return null

  return (
    <a href={post.url}>
      <strong>{post.title}</strong> · <time>{post.date}</time>
      {post.summary && <p>{post.summary}</p>}
    </a>
  )
}
```
