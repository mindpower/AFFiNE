# Quick Start: Building Docker Images in Your Fork

This fork includes a custom GitHub Actions workflow to build and deploy AFFiNE server Docker images to **your own** GitHub Container Registry, without affecting the upstream repository.

## 🚀 Quick Start

### Option 1: Automatic Builds (Easiest)

Docker images are automatically built when you push to:
- `main` branch
- Any `claude/**` branches

Images are published to: `ghcr.io/<your-username>/affine:latest`

### Option 2: Manual Builds

1. Go to the **Actions** tab in your GitHub repository
2. Select **"Build Docker Image (Fork)"** workflow
3. Click **"Run workflow"**
4. Choose options and click **"Run workflow"** button

### Option 3: Pull Existing Images

If images have already been built:

```bash
# Replace 'mindpower' with your GitHub username
docker pull ghcr.io/mindpower/affine:latest
docker run -p 3010:3010 ghcr.io/mindpower/affine:latest
```

## 📦 What You Get

After a successful build, you'll have Docker images tagged with:
- `latest` - Most recent build
- `<branch-name>` - Branch-specific builds
- `<commit-sha>` - Specific commit builds

## 📚 Full Documentation

For detailed instructions, configuration options, and troubleshooting, see:
- **[Docker Build Documentation](docs/docker-build-fork.md)**

## 🔧 Running the Server

### Basic Usage

```bash
docker run -d \
  --name affine \
  -p 3010:3010 \
  -e DATABASE_URL="postgresql://user:pass@host:5432/affine" \
  ghcr.io/<your-username>/affine:latest
```

### With Docker Compose

See the full example in [docs/docker-build-fork.md](./docker-build-fork.md#with-docker-compose)

## ✨ Key Features

- ✅ **No upstream impact** - Everything stays in your fork
- ✅ **Automatic builds** - Push and get images automatically
- ✅ **Manual control** - Trigger builds with custom tags anytime
- ✅ **Free hosting** - Uses GitHub Container Registry (GHCR)
- ✅ **No secrets required** - Works out of the box with `GITHUB_TOKEN`

## 🛠️ Workflow File

The workflow is located at: `.github/workflows/build-docker-fork.yml`

Feel free to customize it for your needs!

---

**Need help?** Check the [full documentation](docs/docker-build-fork.md) or view workflow runs in the Actions tab.
