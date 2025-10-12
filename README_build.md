# Rebuilding the Paul-Jo-Dreyer GitHub Pages Site

This document explains how to rebuild the Bookshop + Jekyll site locally and deploy it to GitHub Pages.

---

## **Prerequisites**

- Ubuntu/Debian or macOS
- Git installed
- Build tools and libraries (for native Ruby gems)
- Node.js 20
- Ruby ≥ 3.2

---

## **1. Install dependencies**

### Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y build-essential libssl-dev libreadline-dev zlib1g-dev git curl
```


### Clone rbenv
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
cd ~/.rbenv && src/configure && make -C src

### Add to shell
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc

### Install ruby-build plugin
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

### Install Ruby ≥ 3.2
rbenv install 3.2.2
rbenv global 3.2.2

### Verify
ruby -v

### Install bundler
gem install bundler
bundle -v


## 2. Build
npm run jekyll