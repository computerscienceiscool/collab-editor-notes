# Hugo Sitemaps & Search Engine Setup
## Essential Guide for the CSWG Website

---

## What is Hugo?

Hugo is a **static site generator**. It takes plain text files written in Markdown and converts them into a complete website (HTML, CSS, JavaScript). Unlike WordPress, which builds pages dynamically on every visit, Hugo builds the entire site once, ahead of time.

**Why this matters:**
- **Fast** — No database queries or server processing; just serving files
- **Secure** — Fewer attack vectors without PHP or databases
- **Simple** — Can be hosted anywhere, including GitHub Pages (which CSWG uses)
- **Version controlled** — The entire site lives in a Git repository

---

## How Hugo Builds the Sitemap

### Automatic Generation

Hugo generates a sitemap **automatically** every time you build the site. When you run `hugo` or `make build`:

1. Hugo scans all content pages (everything in `content/`)
2. Excludes pages marked as `draft: true`
3. Creates `/sitemap.xml` with URLs for all published pages
4. Uses the `date` or `lastmod` from each page's front matter

**You don't need to manually create or update the sitemap** — it regenerates with each build.

### What the Sitemap Contains

```xml
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://cswg.infrastructures.org/</loc>
    <lastmod>2025-11-20</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.5</priority>
  </url>
  <url>
    <loc>https://cswg.infrastructures.org/docs/history/</loc>
    <lastmod>2024-02-13</lastmod>
  </url>
</urlset>
```

Each `<url>` entry includes:
- **loc** — The page URL
- **lastmod** — When the page was last modified
- **changefreq** — How often the page typically changes
- **priority** — Relative importance (0.0 to 1.0)

### Recommended Configuration

Add this to `config.toml` to customize sitemap behavior:

```toml
baseURL = 'https://cswg.infrastructures.org/'
languageCode = 'en-us'
title = 'Community Systems Working Group'
theme = "hugo-book"

# Sitemap configuration
[sitemap]
  changefreq = "weekly"
  filename = "sitemap.xml"
  priority = 0.5

# Enable robots.txt generation
enableRobotsTXT = true
```

### Robots.txt

When `enableRobotsTXT = true` is set, Hugo generates a `robots.txt` file that tells search engines where to find your sitemap:

```
User-agent: *
Sitemap: https://cswg.infrastructures.org/sitemap.xml
```

---

## Why Search Engines Need Access to the Sitemap

**Search engines won't automatically find your sitemap.** Even though it's publicly available at `https://cswg.infrastructures.org/sitemap.xml`, you need to tell Google and Bing where it is.

Without a submitted sitemap, search engines:
- May miss new pages entirely
- Won't know when pages are updated
- Take much longer to discover content

With a submitted sitemap:
- New content is discovered faster
- Every page is known to search engines
- You can monitor indexing status and errors

---

## Google Search Console Setup

### What It Does
Google Search Console helps you monitor your site's presence in Google search results, see what queries bring traffic, and identify problems.

### Setup Steps

1. **Go to** https://search.google.com/search-console

2. **Add Property** — Click "Add Property" and choose "URL prefix"
   - Enter: `https://cswg.infrastructures.org/`

3. **Verify Ownership** (HTML File method recommended for GitHub Pages):
   - Google provides a file like `google1234567890abcdef.html`
   - Add this file to the `static/` folder in the Hugo repo
   - Commit and push to GitHub
   - Click "Verify" in Search Console

4. **Submit Sitemap**:
   - Go to "Sitemaps" in the left menu
   - Enter: `sitemap.xml`
   - Click "Submit"

5. **Monitor** — Check back regularly for indexing status and crawl errors

---

## Bing Webmaster Tools Setup

### What It Does
Covers Bing, Yahoo, and DuckDuckGo searches (about 6-9% of search traffic).

### Setup Steps

1. **Go to** https://www.bing.com/webmasters

2. **Sign in** with a Microsoft account

3. **Add Site**:
   - **Easiest**: Import from Google Search Console (if already set up)
   - **Or**: Add manually with `https://cswg.infrastructures.org/`

4. **Verify Ownership** — Similar options to Google (XML file, meta tag, or DNS)

5. **Submit Sitemap**:
   - Go to "Sitemaps" section
   - Submit: `https://cswg.infrastructures.org/sitemap.xml`

---

## What Sitemaps Do and Don't Do

### What They DO:
- Speed up discovery of new content
- Ensure all pages are known to search engines
- Signal which pages are important (via priority)
- Show when content was updated (via lastmod)

### What They DON'T Do:
- Guarantee indexing (search engines decide what to index)
- Directly improve rankings (content quality matters more)
- Replace the need for good content and backlinks

---

## Key URLs

| Resource | URL |
|----------|-----|
| CSWG Site | https://cswg.infrastructures.org/ |
| Sitemap | https://cswg.infrastructures.org/sitemap.xml |
| Robots.txt | https://cswg.infrastructures.org/robots.txt |
| Google Search Console | https://search.google.com/search-console |
| Bing Webmaster Tools | https://www.bing.com/webmasters |

---

## Quick Reference: Hugo Commands

```bash
# Start local development server
hugo server

# Build the site (generates sitemap)
hugo

# Build and deploy (CSWG specific)
make ship
```

---

## Glossary

| Term | Definition |
|------|------------|
| **Sitemap** | XML file listing all pages on a site, used by search engines for discovery |
| **Robots.txt** | File that tells search engine crawlers which pages to access and where to find the sitemap |
| **Indexing** | When a search engine adds a page to its database so it can appear in search results |
| **Crawling** | When a search engine bot visits pages to read their content |
| **Front Matter** | Metadata at the top of a Markdown file (between `---` lines) containing title, date, tags, etc. |
| **Static Site Generator** | Tool that builds HTML files from templates and content, rather than generating pages dynamically |

---

## Checklist

### Immediate Actions
- [x] Verify sitemap exists at `https://cswg.infrastructures.org/sitemap.xml`
- [ ] Check if robots.txt exists at `https://cswg.infrastructures.org/robots.txt`
- [ ] Update `config.toml` with sitemap configuration
- [ ] Ensure baseURL uses `https://` (not `http://`)

### Google Search Console
- [ ] Add property for `https://cswg.infrastructures.org/`
- [ ] Download and add verification HTML file to `static/` folder
- [ ] Verify ownership
- [ ] Submit sitemap URL

### Bing Webmaster Tools
- [ ] Add site (or import from Google)
- [ ] Complete verification
- [ ] Submit sitemap URL

### Ongoing
- [ ] Check Search Console weekly for errors
- [ ] Monitor which search terms bring traffic
