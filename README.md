# My Hugo Site

This repository contains the source code for my static website built with [Hugo](https://gohugo.io/), a fast and flexible static site generator written in Go.

## 🚀 Features

- Built with [Hugo](https://gohugo.io/)
- Uses the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme
- Easy content management with Markdown
- Fast, secure, and easy to deploy


## 🛠️ Getting Started

### Prerequisites

- [Hugo](https://gohugo.io/getting-started/installing/) (v0.128.0 or later recommended)
- [Git](https://git-scm.com/)


### Installation

1. **Clone the repository:**

```bash
git clone https://github.com/matteobisi/msbiro.net.git
cd msbiro.net
```

2. **Start the local development server:**

```bash
hugo server
```

Your site will be available at [http://localhost:1313/](http://localhost:1313/).

### Creating Content

Add new posts or pages using:

```bash
hugo new posts/my-new-post.md
```

Edit the Markdown file in the `content/posts` directory.

## 🖌️ Configuration

- Site configuration is in `config.toml` (or `config.yaml`/`config.json` depending on your setup).
- Theme-specific options can be customized according to the [PaperMod documentation](https://adityatelange.github.io/hugo-PaperMod/docs/).


## 🚢 Deployment

You can deploy your site using [GitHub Pages](https://pages.github.com/) or any static hosting provider.

### Deploy to GitHub Pages (Mono Repo Approach)

1. Build the site:

```bash
hugo
```

This generates the static files in the `public/` directory.
2. Push the contents of `public/` to your `gh-pages` branch or your GitHub Pages repository.

For automated deployment, you can use GitHub Actions. See [this guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) for details.

## 📂 Directory Structure

- `content/` – Your Markdown content
- `themes/` – Hugo themes (e.g., PaperMod)
- `static/` – Static assets (images, favicon, etc.)
- `public/` – Generated site (do not edit directly)
- `config.toml` – Site configuration


## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**References and further reading:**

- [Hugo Quick Start Guide](https://gohugo.io/getting-started/quick-start/)
- [PaperMod Theme Documentation](https://adityatelange.github.io/hugo-PaperMod/docs/)