# quantum-site

A static site generator written in the Quantum language. Write pages and blog posts in Markdown and pick a look with templates; one command turns them into a complete website with a home page, tag pages, an archive, an RSS feed and a sitemap.

```bash
qrun quantum-site-cli.sa init my-blog --title "My Blog"
cd my-blog
qrun ../quantum-site-cli.sa build
# open public/index.html
```

- **Pure Quantum.** No Node, Python or other tools; it runs on `qrun`.
- **Markdown with GitHub extras:** tables, task lists, fenced code, strikethrough, auto links, heading anchors and a table of contents.
- **Handlebars-style templates:** `{{ }}`, `{{#if}}`, `{{#each}}`, partials, layouts that wrap other layouts, and 30 filters.
- **Blog features built in:** posts dated from their file names, tags, an archive, newer/older links, reading time, summaries, drafts, RSS and a sitemap.
- **Links that work anywhere.** Every link is relative, so the output works from a folder on your disk, on GitHub Pages or on any web server.
- **Fast.** On a laptop, a 1,000-post blog builds in about 8 seconds.

## Install

```bash
qpm install quantum-site
```

The package goes into `quantum_qubits/quantum-site/`. The command-line tool is the single file `dist/quantum-site-cli.sa`, so you can also just copy that file next to your site.

> `qrun` below is the Quantum interpreter. If it isn't on your PATH, use its full path, e.g. `C:\QuantumLogicLabs\QuantumLanguage\seed\qrun.exe`.

## Commands

```bash
qrun quantum-site-cli.sa init [folder] [--title "My Blog"]   # starter site; never overwrites files
qrun quantum-site-cli.sa build [--drafts] [--out folder]     # build into public/
qrun quantum-site-cli.sa new "Post title" [--tags a,b]       # content/posts/<today>-post-title.md
qrun quantum-site-cli.sa help
```

Run with no command, it builds the current folder. If there is no `site.json` yet, it creates the starter site first.

## Folder layout

```
my-blog/
├── site.json                    site settings (below)
├── content/                     your pages
│   ├── about.md                 -> public/about.html
│   ├── notes.html               -> public/notes.html (HTML files are templates too)
│   ├── posts/
│   │   └── 2026-11-02-hello.md  -> public/posts/hello.html (a post, dated 2026-11-02)
│   └── images/cat.png           -> copied as is
├── layouts/                     templates
│   ├── base.html                the page shell (head, header, footer)
│   ├── page.html  post.html     how pages and posts look
│   ├── index.html               home page (when content/ has no index.md)
│   ├── tag.html  tags.html      one page per tag, and the list of tags
│   ├── archive.html  404.html
│   └── partials/header.html     used as {{> header}}
└── static/style.css             copied to the root of public/
```

Files in `content/posts/` are blog posts, and every other page is a plain page. Generated pages (home, tags, archive, 404) are made only when their layout exists, so you can delete the ones you don't want.

## site.json

```json
{
  "title": "My Blog",
  "description": "Notes on Quantum and the web.",
  "author": "Ada",
  "baseUrl": "https://ada.github.io/blog",
  "nav": [{"title": "Home", "url": "index.html"}, {"title": "About", "url": "about.html"}],
  "params": {"github": "https://github.com/ada"}
}
```

| Key | Default | |
|---|---|---|
| `title`, `description`, `author`, `language` | | shown by the templates as `site.title` and so on |
| `baseUrl` | `""` | your site's address. It's needed for full links in `feed.xml` and for `sitemap.xml` |
| `nav` | `[]` | menu items `{"title", "url"}` |
| `params` | `{}` | anything else you want in templates: `site.params.github` |
| `contentDir`, `layoutsDir`, `staticDir`, `outDir` | `content`, `layouts`, `static`, `public` | folder names |
| `postsDir` | `posts` | the folder inside `content/` that holds blog posts |
| `drafts` | `false` | build pages that have `draft: true` |
| `feed`, `feedItems`, `sitemap`, `tagPages` | `true`, `20`, `true`, `true` | turn the generated files on or off |
| `summaryWords`, `recentPosts` | `40`, `5` | length of automatic summaries, and size of `site.recent` |

## Front matter

Put a block like this at the top of any page:

```markdown
---
title: Hello, world
date: 2026-11-02
tags: [quantum, web]
description: Shown in lists and in the page's meta description.
layout: post
draft: false
---

The page content, in **Markdown**.
```

- `title`: defaults to a `# Heading` on the first line (which is then removed from the body), then to the file name.
- `date`: defaults to the `YYYY-MM-DD-` prefix of the file name.
- `slug`: changes the output file name. `permalink: /about/me/` sets the whole path, so this page goes to `about/me/index.html`.
- `layout`: defaults to `post` for posts and `page` for everything else. `layout: none` writes the content without any layout.
- Any other key becomes available in templates as `page.<key>`.

The format is a YAML subset: strings (quoted or not), numbers, `true`/`false`, `null`, `[inline, lists]`, `- block` lists and one level of nested keys.

## Templates

```handlebars
<h1>{{ page.title }}</h1>                         {{! escaped }}
{{{ content }}}                                   {{! raw HTML }}
<p>{{ page.date | date "long" }} · {{ page.readingTime }} min read</p>

{{#each site.posts | limit 5}}
  <a href="{{ url | rel }}">{{ title }}</a>{{#unless @last}}, {{/unless}}
{{else}}
  No posts yet.
{{/each}}

{{#if page.tags}} ... {{else if page.draft}} ... {{else}} ... {{/if}}
{{#if page.layout == "post" and not page.hideDate}} ... {{/if}}
{{#each site.posts as post}} {{ post.title }} {{/each}}
{{#with page.author}} {{ name }} {{/with}}
{{> header}}   {{> post-card post}}
```

- **Layouts wrap each other:** a layout with `layout: base` in its own front matter is placed inside `base.html` as `{{{ content }}}`.
- **Name lookup:** names are searched from the innermost block outwards, so `{{ site.title }}` works anywhere. `@root` is the top level.
- **Inside `#each`:** you get `@index`, `@number` (counting from 1), `@first`, `@last` and, for dicts, `@key`.
- **Whitespace and literals:** `{{~ x ~}}` trims the whitespace around a tag, and `\{{` prints literal braces.

**Filters**

| Filter | What it does |
|---|---|
| `upper` `lower` `capitalize` `trim` | change text |
| `default x` | gives `x` when the value is empty |
| `date "long"` | formats a date; styles: `long` (2 November 2026), `short`, `us`, `iso`, `month`, `year`, `rfc822` |
| `slug` `truncate n` `words n` `striptags` `escape` `wordcount` | text helpers |
| `markdown` `inline` | render Markdown (use them inside `{{{ }}}`) |
| `length` `first` `last` `join ", "` `limit n` `offset n` `reverse` | list helpers |
| `sort "field" "desc"` `where "field" value` | sort and filter lists of pages |
| `plus n` `minus n` `append s` `prepend s` `replace a b` `plural "post" "posts"` `json` | small extras |
| `rel` | makes a site path relative to the current page: `{{ "style.css" \| rel }}` gives `../style.css` inside `posts/` |
| `abs` | puts `baseUrl` in front of a path |

**What templates can use**

| | |
|---|---|
| `site.*` | everything in `site.json`, plus `posts` (newest first), `pages`, `tags` (`name`, `slug`, `url`, `count`, `posts`), `years` (`year`, `posts`), `recent`, `postCount`, `year`, `buildDate`, `generator` |
| `page.*` | `title`, `date`, `url`, `root`, `slug`, `section`, `tags`, `tagLinks`, `summary`, `content`, `toc`, `headings`, `wordCount`, `readingTime`, `isPost`, `isHome`, `newer`, `older` (on posts), and every front matter key |
| generated pages | `page.posts` (home, tag and archive pages), `page.tag` (tag pages), `page.tagList` (tags page), `page.years` (archive) |
| `content` | the rendered page, inside layouts |

## Markdown

**Blocks:** headings with anchor ids, paragraphs, hard line breaks (two spaces or `\`), fenced code with a language class, indented code, block quotes, nested ordered and unordered lists, task lists, tables with column alignment, horizontal rules, raw HTML blocks, and `[label]: url` reference definitions.

**Inline:** `**bold**`, `*italic*`, `***both***`, `~~strike~~`, `` `code` ``, `[links](url "title")`, `![images](src)`, `[reference][links]`, `<https://autolinks>`, bare `https://` links, backslash escapes and inline HTML.

## Using the library

The modules are ordinary globals. Bundle them with [quantum-bundle](https://github.com/QuantumLogicsLabs/qpmExamples/tree/main/quantum-bundle) (`# @use quantum-site`), or put `dist/quantum-site.sa` in front of your script:

```python
# @use quantum-site

print(Markdown.render("# Hi\n\nSome *text*."))
print(Template.render("{{#each items}}<li>{{ this }}</li>{{/each}}", {"items": ["a", "b"]}))

let doc = FrontMatter.parse(read_file("post.md"))     # {"data": {...}, "body": "..."}
let report = Site.build({"root": "my-blog", "drafts": true})
print(Site.summary(report))
```

| Global | API |
|---|---|
| `Site` | `build(options)` · `summary(report)` · `loadConfig(root)` · `newPost(root, title, tags)` |
| `Markdown` | `render(text)` · `renderInline(text)` · `headings(text)` · `toc(text, minLevel, maxLevel)` |
| `Template` | `render(source, data)` · `compile(source, name)` · `renderCompiled(tpl, data)` · `registerPartial(name, source)` · `addFilter(name, fn(value, args))` |
| `FrontMatter` | `parse(text)` · `stringify(data, keys)` |
| `SiteUtil` | `slugify` · `escapeHtml` · `stripTags` · `formatDate` · `today` · `listFiles` · `copyFile` · `ensureDir` and more |
| `SiteStarter` | `create(folder, {"title"})` · `files()` |

## Publishing your site on GitHub Pages

1. In `site.json`, set `"outDir": "docs"` and set `"baseUrl"` to your Pages address.
2. Build, then commit and push the repository.
3. On GitHub, open **Settings → Pages**, choose **Deploy from a branch**, then pick `main` and `/docs`.

The build writes `.nojekyll` and `404.html` for you.

## Notes

- Builds don't delete files: a page you remove stays in `public/` until you delete it there.
- Big sites are fine. qrun normally stops scripts after 200 million steps, so `Site.build` lifts that limit for the build.
- Files are read and written as UTF-8, and `.md` files may use Windows (CRLF) line endings.

## Development

```
index.sa                 bundle entry (lists the modules in order)
src/                     util, frontmatter, markdown, template, site, starter
bin/quantum-site.sa      command-line tool
dist/                    pre-bundled single files: library and CLI
test/all.test.sa         145 tests; test/run-tests.sa is the bundled runner
```

```bash
qrun test/run-tests.sa
```

## License

MIT © 2026 Muhammad Bilal Raza
