# yuvaravii.github.io

Personal portfolio and blog — built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

**Live at:** [https://yuvaravii.github.io](https://yuvaravii.github.io)

## Local Development

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/yuvaravii/yuvaravii.github.io.git
cd yuvaravii.github.io

# Run local server
hugo server -D

# Create new blog post
hugo new blog/my-post-title.md

# Create new project page
hugo new projects/my-project.md
```

## Deployment

Automatic via GitHub Actions. Push to `main` → builds → deploys to GitHub Pages.