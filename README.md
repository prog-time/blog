# ProgTime

A blog about programming and technology, built with [Jekyll](https://jekyllrb.com/).

## Stack

- **Jekyll** 4.4
- **Theme**: Minima 2.5
- **Plugins**: jekyll-feed

## Structure

```
├── _posts/         # Blog posts
├── _projects/      # Portfolio projects
├── _jobs/          # Work experience
├── _layouts/       # Custom layouts
├── _includes/      # Reusable components
├── assets/         # CSS, JS, fonts
├── images/         # Image files
├── about.html      # About page
├── projects.html   # Projects page
└── _config.yml     # Site configuration
```

## Local Development

**Requirements**: Ruby, Bundler

```bash
# Install dependencies
bundle install

# Start local server
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

## Adding Content

### New post

Create a file in `_posts/YEAR/YYYY-MM-DD-title.md`:

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
---

Content here.
```

### New project

Create a file in `_projects/`:

```yaml
---
layout: project
title: "Project Name"
---

Description here.
```

### New job

Create a file in `_jobs/`:

```yaml
---
layout: job
title: "Company Name"
---

Description here.
```

## Links

- GitHub: [prog-time](https://github.com/prog-time)
- Telegram: [@iliyalyachuk](https://t.me/iliyalyachuk)
