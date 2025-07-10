# Setting up Giscus Comments

To enable comments on your blog posts, you need to configure Giscus:

## Steps:

1. **Enable GitHub Discussions** on your blog repository:
   - Go to your repo Settings → Features
   - Check "Discussions"

2. **Install the Giscus app**:
   - Visit https://github.com/apps/giscus
   - Install it on your blog repository

3. **Get your configuration**:
   - Visit https://giscus.app
   - Enter your repository: `ryanrasti/blog`
   - Choose your preferences:
     - Page ↔️ Discussions Mapping: `pathname` (already set)
     - Discussion Category: `General` or create a `Blog Comments` category
     - Features: Enable reactions (already set)
     - Theme: `preferred_color_scheme` (already set)

4. **Update the component**:
   - Copy the `data-repo-id` and `data-category-id` from the generated script
   - Update `/src/components/Giscus.astro` with these values

## Current Settings:
- Mapping: pathname (each blog post gets its own discussion)
- Position: Top (comment box above existing comments)
- Theme: Follows user's system preference
- Reactions: Enabled
- Loading: Lazy (better performance)

The component is already added to all blog posts via the BlogPost layout.