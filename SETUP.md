# Setup Guide

This guide will help you set up the Android Interview Preparation repository on your local machine.

## 📋 Prerequisites

Before you begin, ensure you have:

- **Git** installed and configured
- **Ruby** (2.7 or later)
- **Bundler** gem installed
- **Basic knowledge** of Jekyll (optional but helpful)

## 🚀 Installation Steps

### 1. Clone the Repository

```bash
# Clone the repository
git clone https://github.com/your-username/android-interview.git

# Navigate to the project directory
cd android-interview
```

### 2. Install Ruby Dependencies

```bash
# Install required gems
bundle install
```

If you don't have Bundler installed:

```bash
# Install bundler
gem install bundler

# Then install dependencies
bundle install
```

### 3. Build and Serve Locally

```bash
# Serve the site locally
bundle exec jekyll serve
```

Or with specific configuration:

```bash
# Serve with watch mode for live reload
bundle exec jekyll serve --watch

# Serve with draft posts
bundle exec jekyll serve --drafts

# Serve on custom port
bundle exec jekyll serve --port 4001
```

### 4. Access the Site

Open your browser and navigate to:

```
http://localhost:4000
```

## 📁 Project Structure

```
android-interview-prep/
├── _config.yml              # Jekyll configuration
├── _includes/               # Reusable includes
│   └── head-custom.html    # Custom head elements
├── docs/                   # Documentation files
│   ├── introduction.md
│   ├── android-basics.md
│   └── ... (all topics)
├── index.md                # Homepage
├── README.md               # Project readme
├── Gemfile                 # Ruby dependencies
└── LICENSE                 # License file
```

## 🔧 Troubleshooting

### Issue: Bundle install fails

**Solution:**
```bash
# Update Ruby
ruby --version

# Update gems
gem update --system

# Clean and install
bundle clean --force
bundle install
```

### Issue: Jekyll serve doesn't work

**Solution:**
```bash
# Check Jekyll installation
bundle exec jekyll --version

# Regenerate
bundle exec jekyll clean
bundle exec jekyll build
bundle exec jekyll serve
```

### Issue: Port 4000 already in use

**Solution:**
```bash
# Use a different port
bundle exec jekyll serve --port 4001
```

### Issue: Missing dependencies

**Solution:**
```bash
# Reinstall everything
rm Gemfile.lock
bundle install
```

## 🌐 GitHub Pages Setup

### Option 1: Automatic GitHub Pages

1. Push repository to GitHub
2. Go to Settings → Pages
3. Select source branch (usually `main` or `gh-pages`)
4. GitHub will build and publish automatically

### Option 2: Manual Setup with Actions

Create `.github/workflows/jekyll.yml`:

```yaml
name: Jekyll site CI

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.1'
        bundler-cache: true
    - run: bundle exec jekyll build
```

## 📝 Development Workflow

### Making Changes

1. **Create a branch:**
   ```bash
   git checkout -b feature/my-feature
   ```

2. **Make changes to files**

3. **Test locally:**
   ```bash
   bundle exec jekyll serve
   ```

4. **Commit changes:**
   ```bash
   git add .
   git commit -m "Add: Description of changes"
   ```

5. **Push and create PR:**
   ```bash
   git push origin feature/my-feature
   ```

### Preview Changes

Always preview your changes locally before pushing:

```bash
# Build the site
bundle exec jekyll build

# Serve locally
bundle exec jekyll serve

# Check for errors
bundle exec jekyll doctor
```

## 🛠️ Useful Commands

```bash
# Clean build artifacts
bundle exec jekyll clean

# Build site
bundle exec jekyll build

# Serve with live reload
bundle exec jekyll serve --livereload

# Check for issues
bundle exec jekyll doctor

# Build for production
JEKYLL_ENV=production bundle exec jekyll build
```

## 🔍 Validating Your Setup

Run these commands to verify your setup:

```bash
# Check Ruby version
ruby -v

# Check Bundler
bundle -v

# Check Jekyll
bundle exec jekyll -v

# Check site for issues
bundle exec jekyll doctor
```

## 📚 Additional Resources

- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [Ruby Installation Guide](https://www.ruby-lang.org/en/documentation/installation/)

## 🆘 Getting Help

If you encounter issues:

1. Check the troubleshooting section above
2. Search existing issues on GitHub
3. Create a new issue with details:
   - Your operating system
   - Ruby version (`ruby -v`)
   - Bundler version (`bundle -v`)
   - Error messages
   - Steps to reproduce

## ✅ Quick Start Checklist

- [ ] Ruby installed (2.7 or later)
- [ ] Bundler installed
- [ ] Repository cloned
- [ ] Dependencies installed (`bundle install`)
- [ ] Site running locally (`bundle exec jekyll serve`)
- [ ] Can access http://localhost:4000
- [ ] Ready to make changes

---

**Need help?** Open an issue or reach out to the maintainers!

