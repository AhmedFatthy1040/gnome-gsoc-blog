# GNOME GSoC 2025 Blog

📝 Blog for my Google Summer of Code 2025 journey with GNOME, including progress updates, technical deep dives, and reflections.

🌐 **Live Blog**: [https://ahmedfatthi.pages.dev/](https://ahmedfatthi.pages.dev/)

## 🚀 Quick Start

### Local Development

1. **Install Hugo** (if not already installed):
   ```bash
   # Windows (using Chocolatey)
   choco install hugo-extended
   
   # Or download from https://github.com/gohugoio/hugo/releases
   ```

2. **Clone and setup**:
   ```bash
   git clone <your-repo-url>
   cd gnome-gsoc-blog
   git submodule update --init --recursive
   ```

3. **Run locally**:
   ```bash
   hugo server -D
   ```
   
   Open http://localhost:1313 in your browser.

### Adding New Posts

```bash
hugo new content posts/your-post-name.md
```

## 🌐 Deployment on Cloudflare Pages

### Automatic Deployment

1. **Connect to Cloudflare Pages**:
   - Go to [Cloudflare Pages](https://pages.cloudflare.com/)
   - Connect your GitHub repository
   - Choose "Hugo" as the framework preset

2. **Build Configuration**:
   - **Build command**: `hugo --minify`
   - **Build output directory**: `public`
   - **Root directory**: `/` (leave empty)

3. **Environment Variables**:
   - `HUGO_VERSION`: `0.147.4`
   - `NODE_VERSION`: `18`

### Manual Deployment

```bash
# Build the site
hugo --minify

# The public/ directory contains your built site
# Upload the contents to your hosting provider
```

## 📝 Content Structure

```
content/
├── posts/           # Blog posts
│   └── introduction.md
├── about.md         # About page
└── _index.md       # Homepage content (optional)
```

## 🎨 Customization

### Theme Configuration

The blog uses the PaperMod theme. Key configuration options in `hugo.toml`:

- **Site info**: Update `title`, `description`, `author`
- **Social links**: Update the `[[params.socialIcons]]` sections
- **Menu**: Customize the navigation menu in `[menu]`
- **Features**: Enable/disable features like reading time, breadcrumbs, etc.

### Adding Your Information

Update these sections in `hugo.toml`:
- Change the `baseURL` to your actual domain
- Update social media links
- Customize the home page welcome message
- Add your project-specific details

## 📊 Features

- ✅ **Fast & Secure**: Built with Hugo static site generator
- ✅ **Responsive Design**: Works on all devices
- ✅ **SEO Optimized**: Meta tags, Open Graph, structured data
- ✅ **Syntax Highlighting**: Code blocks with language detection
- ✅ **Dark/Light Theme**: Automatic theme switching
- ✅ **Search**: Built-in search functionality
- ✅ **RSS Feed**: Automatic RSS generation
- ✅ **Performance**: Optimized for Cloudflare Pages CDN

## 🔧 Development Commands

```bash
# Start development server with drafts
hugo server -D

# Build for production
hugo --minify

# Create new post
hugo new content posts/my-new-post.md

# Update theme
git submodule update --remote themes/PaperMod
```

## 📁 Project Structure

```
gnome-gsoc-blog/
├── content/          # Markdown content files
├── static/           # Static assets (images, files)
├── themes/           # Hugo themes (PaperMod)
├── public/           # Generated site (git ignored)
├── hugo.toml         # Hugo configuration
├── _headers          # Cloudflare Pages headers
├── _redirects        # Cloudflare Pages redirects
└── README.md         # This file
```

## 🤝 Contributing

This is a personal blog for my GSoC journey, but I welcome:
- Suggestions for improvements
- Bug reports
- Technical discussions

## 📧 Contact

- **GitHub**: [AhmedFatthy1040](https://github.com/AhmedFatthy1040)
- **LinkedIn**: [Ahmed Fatthi Al-Khateeb](https://www.linkedin.com/in/ahmedfatthi1040/)

## 📜 License

This blog content is licensed under [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/).

The Hugo theme (PaperMod) is licensed under the MIT License.

---

*Built with ❤️ using Hugo and deployed on Cloudflare Pages*
