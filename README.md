# calmecac.dev

Sitio web oficial de Calmécac. Construido con Astro, Svelte y desplegado en Cloudflare Pages.

## Stack

- [Astro](https://astro.build) — framework principal
- [Svelte](https://svelte.dev) — componentes interactivos (theme toggle, lang toggle)
- [Cloudflare Pages](https://pages.cloudflare.com) — hosting y deploy

## Estructura

```
src/
  content/
    blog/           ← posts en español (.md)
    blog/en/        ← posts en inglés (.md)
  layouts/
    Base.astro      ← layout principal con header y footer
  components/
    PostCard.astro  ← tarjeta de post para listados
    ThemeToggle.svelte
    LangToggle.svelte
  pages/
    index.astro     ← home (es)
    blog/           ← blog (es)
    projects.astro
    contact.astro
    en/             ← todas las páginas en inglés
  styles/
    global.css      ← tokens, tipografía, dark mode
```

## Desarrollo local

```bash
npm install
npm run dev
```

## Agregar un post

Crea un archivo `.md` en `src/content/blog/` con el siguiente frontmatter:

```markdown
---
title: "Título del post"
description: "Descripción corta para SEO y listados."
date: 2026-03-25
tags: ["tag1", "tag2"]
readingTime: 8
---

Contenido del post en Markdown...
```

Para la versión en inglés, crea el mismo archivo en `src/content/blog/en/`.

## Deploy

El sitio se despliega automáticamente en Cloudflare Pages al hacer push a `main`.

Configura el proyecto en Cloudflare Pages con:
- Build command: `npm run build`
- Build output: `dist`
- Node version: `20`
