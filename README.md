# 🍎 iOS CI/CD Mac Setup Guide

This document explains how to set up **local Mac** or **Mac CI server (Mac Mini)** for building and deploying the iOS app using **Fastlane + GitLab CI**.

---

## 1. Supported Environment

| Component | Requirement |
|--------|------------|
| macOS | 13+ (Ventura or newer recommended) |
| Xcode | Installed from App Store |
| Ruby | **3.1.7** (managed via rbenv) |
| Package Manager | Homebrew |
| CI | GitLab Runner (shell executor) |

---

## 2. Xcode Setup

### Install Xcode
- Install via **Mac App Store**
- Open once after install

### Accept License
```bash
sudo xcodebuild -license accept
```

### Select Command Line Tools
```bash
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
```

### Verify:
```bash
xcodebuild -version
```

## 3. Homebrew
### Install Homebrew (if not installed):
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```


### Update:
```bash
brew update
```

## 4. Ruby Environment (rbenv)
### Install rbenv & ruby-build
```bash
brew install rbenv ruby-build
```

### Initialize rbenv
Add to shell profile (~/.zshrc or ~/.bashrc):
```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

Reload shell:
```bash
source ~/.zshrc
```

### Install Ruby Version (Project Standard)
```bash
rbenv install 3.1.7
rbenv global 3.1.7
rbenv rehash
```

### Verify:
```bash
ruby -v
# ruby 3.1.7p261
```

## 5. Project Dependencies
### Bundler
```bash
gem install bundler
```

### Install Gems
From project root:
```bash
bundle install
```

### CocoaPods
```bash
bundle exec pod install
```

## 6. Fastlane Setup

Fastlane is installed via Bundler (recommended).

### Verify
```bash
bundle exec fastlane --version
```

### Available Lanes
| Lane        | Description                  |
| ----------- | ---------------------------- |
| `ios beta`  | Build + upload to TestFlight |


### Run locally:
```bash
bundle exec fastlane ios beta
```

## 7. Environment Variables
### Required Variables
These must be set locally or via GitLab CI Variables:

| Variable                            | Description                     |
| ----------------------------------- | ------------------------------- |
| `APP_IDENTIFIER`                    | App bundle identifier           |
| `APP_IDENTIFIER_MAIN`               | Main app identifier             |
| `APP_IDENTIFIER_CONTENT`            | Notification Content Extension  |
| `APP_IDENTIFIER_SERVICE`            | Notification Service Extension  |
| `APP_STORE_CONNECT_API_KEY_ID`      | ASC API Key ID                  |
| `APP_STORE_CONNECT_ISSUER_ID`       | ASC Issuer ID                   |
| `APP_STORE_CONNECT_API_KEY_CONTENT` | Base64 encoded `.p8` key        |
| `DISCORD_TESTFLIGHT_WEBHOOK_URL`    | (Optional) Discord notification |


## 8. GitLab Runner (Mac Server Only)
### Install Runner
```bash
brew install gitlab-runner
```

### Start Runner
```bash
brew services start gitlab-runner
```

### Register Runner
```bash
gitlab-runner register
```

## 9. CI/CD Trigger Rules

CI will run only if:

✅ Branch starts with:
```bash
dev/*
```

✅ Commit message contains:
```bash
| build:ios-dev
```

❌ CI will NOT run on:
- Merge Requests
- Other branches
- Commits without trigger keyword

Example valid commit:
```bash
[stock]<fix>: Fix mapping issue | build:ios-dev
```

## 10. Build Artifacts

Generated files:
```bash
Builds/
- ├── AppName-Dev-(version)(build).ipa
- ├── AppName-Dev-(version)(build).dSYM.zip
```
Artifacts are stored for 7 days in GitLab.

## 11. Notes & Best Practices
- Do not install Fastlane globally
- Always use bundle exec
- Avoid committing secrets
- Use ASC API Key (not Apple ID login)
- Keep build logic inside Fastlane, not CI YAML
- Ensure certificates & provisioning profiles are installed.

## 12. Maintainers
CI/CD Owner: @anjas @fadilah @lukius @arnold @agus
