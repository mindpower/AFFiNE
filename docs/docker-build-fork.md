# Building Docker Images in Your Fork

This document explains how to build and deploy Docker images for the AFFiNE server in your forked repository without affecting the upstream repository.

## Quick Start

### Method 1: Using GitHub Actions (Recommended)

The workflow `.github/workflows/build-docker-fork.yml` has been created to automatically build and push Docker images to your GitHub Container Registry.

#### Automatic Builds

The workflow automatically triggers on:
- Push to `main` branch
- Push to any `claude/**` branches

#### Manual Builds

You can also trigger builds manually with custom options:

1. Go to your repository on GitHub
2. Click on **Actions** tab
3. Select **Build Docker Image (Fork)** workflow
4. Click **Run workflow**
5. Configure options:
   - **Branch**: Select the branch to build from
   - **Docker image tag**: Custom tag (e.g., `v1.0.0`, `dev`, `test`)
   - **Build web/admin/mobile**: Toggle frontend builds on/off

#### Accessing Your Images

After the workflow completes, your Docker images will be available at:
```
ghcr.io/<your-github-username>/affine:latest
ghcr.io/<your-github-username>/affine:<branch-name>
ghcr.io/<your-github-username>/affine:<commit-sha>
```

For example, if your GitHub username is `mindpower`:
```bash
# Pull the latest image
docker pull ghcr.io/mindpower/affine:latest

# Pull a specific branch build
docker pull ghcr.io/mindpower/affine:claude-deploy-docker-image-for-server

# Pull a specific commit
docker pull ghcr.io/mindpower/affine:655cc88
```

### Method 2: Local Build

If you prefer to build locally:

```bash
# 1. Build all frontend components
yarn affine @affine/web build
yarn affine @affine/admin build
yarn affine @affine/mobile build

# 2. Build server native modules (requires Rust toolchain)
yarn workspace @affine/server-native build

# 3. Build server
yarn workspace @affine/server build

# 4. Install production dependencies
yarn config set --json supportedArchitectures.cpu '["x64"]'
yarn workspaces focus @affine/server --production

# 5. Generate Prisma client
yarn workspace @affine/server prisma generate

# 6. Move node_modules
mv ./node_modules ./packages/backend/server

# 7. Build Docker image
docker build -f .github/deployment/node/Dockerfile -t affine-server:local .

# 8. Tag and push to your registry
docker tag affine-server:local ghcr.io/<your-username>/affine:latest
docker push ghcr.io/<your-username>/affine:latest
```

## Running Your Docker Image

### Basic Usage

```bash
docker run -d \
  --name affine-server \
  -p 3010:3010 \
  -e DATABASE_URL="postgresql://user:password@host:5432/affine" \
  -e REDIS_SERVER_HOST="redis-host" \
  ghcr.io/<your-username>/affine:latest
```

### With Docker Compose

Create a `docker-compose.yml`:

```yaml
version: '3.8'

services:
  affine:
    image: ghcr.io/<your-username>/affine:latest
    container_name: affine-server
    ports:
      - "3010:3010"
    environment:
      - DATABASE_URL=postgresql://affine:affine@postgres:5432/affine
      - REDIS_SERVER_HOST=redis
      - NODE_ENV=production
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    container_name: affine-postgres
    environment:
      - POSTGRES_USER=affine
      - POSTGRES_PASSWORD=affine
      - POSTGRES_DB=affine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: affine-redis
    restart: unless-stopped

volumes:
  postgres_data:
```

Then run:
```bash
docker-compose up -d
```

## Configuration Options

### Workflow Customization

Edit `.github/workflows/build-docker-fork.yml` to customize:

#### Enable Multi-Architecture Builds

Uncomment the additional architectures in the workflow:

```yaml
# In build-server-native job, uncomment:
- name: aarch64-unknown-linux-gnu
  file: server-native.arm64.node
- name: armv7-unknown-linux-gnueabihf
  file: server-native.armv7.node

# In build-docker job, change platforms to:
platforms: linux/amd64,linux/arm64,linux/arm/v7
```

#### Add Environment Variables

If you need additional build-time configuration (Sentry, analytics, etc.), add them to the build steps:

```yaml
- name: Build Web
  run: yarn affine @affine/web build
  env:
    BUILD_TYPE: 'stable'
    SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
    # Add more as needed
```

### Required Secrets

The workflow uses only the built-in `GITHUB_TOKEN` for pushing to GHCR. No additional secrets are required for basic functionality.

Optional secrets you can add in **Settings → Secrets and variables → Actions**:
- `SENTRY_AUTH_TOKEN` - For error tracking
- `R2_*` - For Cloudflare R2 storage
- `CAPTCHA_SITE_KEY` - For CAPTCHA integration

## Troubleshooting

### Images Not Showing in Packages

1. Ensure GitHub Container Registry is enabled in your fork
2. Check package visibility settings at: `https://github.com/<username>?tab=packages`
3. Make the package public if needed: Package settings → Change visibility → Public

### Build Failures

**Node.js version mismatch:**
- The workflow uses the version specified in `.nvmrc` (22.22.3)
- Ensure your local environment matches this version

**Rust toolchain issues:**
- Install Rust: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- The workflow uses the version specified in `rust-toolchain.toml`

**Out of disk space:**
- GitHub Actions runners have limited disk space
- The workflow only builds x64 by default to save space
- Enable multi-arch only if needed

### Permission Denied

If you get permission errors when pulling images:

```bash
# Login to GHCR
echo $GITHUB_PAT | docker login ghcr.io -u <username> --password-stdin

# Or use GitHub CLI
gh auth token | docker login ghcr.io -u <username> --password-stdin
```

## Differences from Upstream

This fork's Docker workflow differs from the upstream in:

1. **Registry**: Pushes to `ghcr.io/<your-username>/affine` instead of `ghcr.io/toeverything/affine`
2. **Triggers**: Can be manually triggered with custom options
3. **Architecture**: Builds x64 only by default (multi-arch is optional)
4. **Secrets**: Works without additional secrets (upstream requires Sentry, R2, etc.)
5. **Environment**: Uses `stable` build type instead of environment-specific configs

## Additional Resources

- [GitHub Container Registry Documentation](https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Docker Documentation](https://docs.docker.com/)
- [AFFiNE Documentation](https://docs.affine.pro/)

## Support

For issues specific to your fork:
- Check the Actions tab for build logs
- Review the workflow file for configuration issues

For AFFiNE-specific issues:
- Visit the [upstream repository](https://github.com/toeverything/AFFiNE)
- Check the [official documentation](https://docs.affine.pro/)
