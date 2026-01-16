---
title: Blog migration report
date: 2026-01-16 13:18:00 +0100
description: "Blog migration to Hugo report, new changes and future plans"
tags:
- blog
showComments: true
---

Welcome back friend. As you can see I have finally updated my blog, I migrated ti from Jekyll to Hugo. This is a change I have been planning for a while and I finally got around to it. I have been using Jekyll for 10 years now and it has served me well, but I have been meaning to try Hugo for a while and I finally got around to it. Hugo provides faster build times, have module system and asset pipeline and provides a lot of features out of the box, including a native GitHub Actions workflow for deployment to GitHub Pages. For me GitHub Pages is still the best option for hosting a static site, and I do not plan to change that anytime soon. And this migration helped me to learn in detail how Hugo works, and how to use it to create a static site.

If you have been around since I started this blog, you will remember this the second migration I have done, the first one was from [Wordpress to Jekyll]({{< ref "posts/2016-01-04-blog-migration-done-let-the-new-era-begin.md" >}}). That was back in 2016 and it was a lot of work, but it was worth it. It took more than a mont at the time of planning, testing the site in GitHub Pages, review the posts one by one manually, and finally getting around to it. Along these 10 years I have done many changes, updated the Minimal Mistakes theme, added Google Analytics, Disqus, updated the colors, fonts, etc.

This time the migration took me 2 hours, I planned for it many times, investigated many possible approaches, and it looked like it was going to be a lot of work. However I used Claude Code, I started by analyzing the repo structure and provide me with a plan that I was intending toi follow manually word by word but instead I gave Claude semi-full autonomy to make the migration and it was a great experience.

Claude went through the blog Jekyll structure, explain the differences with Hugo and propose to create a new directory to contain the new version in Hugo, that way I will always have the original jekyll version if something goes wrong. Before executing each task I reviewed and gave Claude the green light to proceed, I am using Claude and Gemini CLI these days in more autonomous ways but this time I was more hands-on to make sure everything was done correctly, in the end this blog is a reflection of my work and I did not want to risk anything.

After ten years my jekyll site had a few problems, like a tag and category taxonomy that was confusing with many tags and categories overlapping with each other, and it was not easy to navigate. SO i used Claude to analyze it and suggest a new taxonomy that was much better and easier to navigate and to remove the Categories from all posts. This help me a lot to learn the Hugo taxonomy system and to understand how to use it.

Below is the detail of the migration, I used Claude as well to help me generate it, that way I can share it here and save it on my notes and my lab wiki.

## Migration Process

### Phase 1: Initial Migration

The core migration involved:

1. **Theme Setup**
   - Replaced Jekyll/Minimal Mistakes with Hugo/Congo theme.
   - Added Congo as a Git submodule in `themes/congo`.
   - Created Hugo configuration structure under `config/_default/`:
     - `hugo.toml` - Main site configuration.
     - `params.toml` - Theme parameters.
     - `menus.en.toml` - Navigation menus.
     - `languages.en.toml` - Language settings.
     - `markup.toml` - Markdown rendering options.
     - `module.toml` - Hugo modules configuration.

2. **Content Migration**
   - Moved all 223 posts from `_posts/` to `content/posts/`.
   - Converted front matter from Jekyll format to Hugo format.
   - Migrated About page from `_pages/about.md` to `content/about/index.md`.
   - Created default archetype in `archetypes/default.md`.

3. **Front Matter Conversion**
   - Preserved `title`, `date`, `tags` fields.
   - Added `showComments: true` for Disqus integration.
   - Removed Jekyll-specific fields (`layout`, `author`, `header`, `toc`, etc.).
   - Converted Jekyll `excerpt` to Hugo `description` where applicable.

4. **URL Structure Preservation**
   - Configured permalinks to match Jekyll's URL pattern: `/:year/:month/:day/:slug/`.
   - This ensures all existing links and search engine rankings remain valid.

5. **Deployment Configuration**
   - Created GitHub Actions workflow (`.github/workflows/hugo.yml`).
   - Configured Hugo Extended v0.154.5 for Sass/SCSS processing.
   - Set up automatic deployment to GitHub Pages on push to master.

6. **Features Configured**
   - Disqus comments (shortname: `juanmasblog`).
   - Google Analytics.
   - Dark theme as default with auto-switch capability.
   - Search functionality enabled.
   - Code copy button enabled.
   - Social sharing links (X/Twitter, LinkedIn, Reddit, Email, Bluesky).
   - Reading time display.
   - Table of contents for articles.

### Phase 2: UI Refinements

- Moved dark/light appearance switcher from footer to header menu.
- Added custom favicons partial to use existing `favicon.ico`.

### Phase 3: Homepage Improvements

- Created custom `article-link.html` partial for homepage.
- Configured to show post `description` field instead of auto-generated summary.
- Added descriptions to recent posts for better homepage display.
- Simplified homepage post listing layout.

### Phase 4: Taxonomy Cleanup

Major cleanup of tags and categories:

1. **Removed Categories**
   - Deleted all categories from 223 posts.
   - Removed category taxonomy from Hugo config.
   - Removed Categories from navigation menu.

2. **Tag Consolidation**
   - Consolidated 250+ tags down to 30 normalized tags
   - Grouped tags by domain:
     - **Cloud:** azure, vmware, openstack.
     - **Containers:** kubernetes, containers, cloud-native, docker, aks.
     - **Operating Systems:** linux, hp-ux, bsd, windows.
     - **Infrastructure:** virtualization, networking, storage, security.
     - **Practices:** sysadmin, devops, scripting.
     - **Hardware:** hp-servers, homelab.
     - **VMware Stack:** esxi, vsphere, nsx.
     - **Other:** azure-devops, vault, career, opinion, blog, personal.

### Phase 5: Cleanup

- Removed duplicate images from `assets/images/` folder.
- Images were duplicated in both `assets/images` and `static/images`.
- Consolidated to `static/images` for content.
- Kept only `author.jpg` in `assets/images` for Hugo's resource pipeline.

## Directory Structure Changes

Here you can see Hugo directory structure and how it compares to Jekyll:

```plaintext
Before (Jekyll):                    After (Hugo):
├── _config.yml                     ├── config/
├── _data/                          │   └── _default/
│   ├── authors.yml                 │       ├── hugo.toml
│   └── navigation.yml              │       ├── params.toml
├── _pages/                         │       ├── menus.en.toml
│   ├── about.md                    │       ├── languages.en.toml
│   └── ...                         │       ├── markup.toml
├── _posts/                         │       └── module.toml
│   └── *.md                        ├── content/
├── assets/                         │   ├── about/
│   └── css/                        │   │   └── index.md
├── Gemfile                         │   └── posts/
└── ...                             │       └── *.md
                                    ├── layouts/
                                    │   └── partials/
                                    ├── static/
                                    │   └── images/
                                    ├── themes/
                                    │   └── congo/
                                    └── archetypes/
```

## GitHub Actions Workflow

As i mentioned I am still usign GitHub Pages for hosting the site, and I have a GitHub Actions workflow that builds the site and deploys it to Pages.

- Triggered on push to master branch.
- Uses Hugo Extended v0.154.5.
- Builds with `--gc --minify` flags.
- Deploys to GitHub Pages using official actions.

## Post-Migration Statistics

- **Total posts migrated:** 223.
- **Tags after consolidation:** 30.
- **Categories:** Removed entirely.
- **Build time improvement:** Significant (Hugo typically builds in seconds vs Jekyll's minutes).
- **All existing URLs preserved:** Yes.

## Lessons Learned

1. **URL preservation is critical** - Keeping the same permalink structure prevents broken links and maintains SEO.
2. **Front matter cleanup** - The migration was a good opportunity to standardize metadata across all posts.
3. **Tag consolidation** - Reducing from 250+ to 28 tags improved navigation and discoverability.
4. **Image organization** - Consolidating images to a single location prevents duplication.
5. **Incremental improvements** - Breaking the migration into phases allowed for testing and refinement.

---

And this is it, migration done and like I said 10 year ago... let the new era begins!

--Juanma
