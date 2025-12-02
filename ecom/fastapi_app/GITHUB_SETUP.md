# GitHub Actions Secrets & Variables Setup Guide

## Required Configuration for CI/CD Pipeline

### Step 1: Generate Docker Hub Token

1. Go to [Docker Hub](https://hub.docker.com/)
2. Click on your profile icon → Account Settings → Security
3. Click "New Access Token"
4. Name it: `github-actions`
5. Select "Read & Write" permissions
6. Click "Generate"
7. Copy the token (you won't see it again)

### Step 2: Configure GitHub Repository

#### A. Add Repository Secrets

1. Go to your GitHub repository
2. Navigate to: **Settings → Secrets and variables → Actions**
3. Click "New repository secret"
4. Add the following secrets:

| Secret Name | Value | Description |
|---|---|---|
| `DOCKERHUB_TOKEN` | Your Docker Hub token from Step 1 | Used to authenticate with Docker Hub for image push |
| `GITHUB_TOKEN` | (Auto-provided) | GitHub's native token, already available in Actions |

#### B. Add Repository Variables

1. In the same **Settings → Secrets and variables → Actions** section
2. Click on the "Variables" tab
3. Click "New repository variable"
4. Add the following variables:

| Variable Name | Value | Description |
|---|---|---|
| `DOCKER_USERNAME` | Your Docker Hub username | Used to tag images on Docker Hub (optional) |

### Step 3: Verify Configuration

To verify secrets are properly set:

```bash
# From repository root
git log --oneline -1
# Should show your latest commit
```

Then trigger a workflow run by pushing to main:

```bash
git add .
git commit -m "chore: update CI/CD pipeline"
git push origin main
```

### Step 4: Monitor First Run

1. Go to your repository
2. Navigate to **Actions** tab
3. Click on the latest workflow run
4. Monitor the following jobs:
   - `Lint and Test`
   - `Build and Push`
   - `Verify Image`

## Expected Workflow Outputs

### Successful Run

```
✓ Lint and Test - PASSED
  ├─ Checkout code
  ├─ Set up Python 3.12
  ├─ Install dependencies
  ├─ Run linting checks
  ├─ Run type checking
  └─ Run tests

✓ Build and Push - PASSED
  ├─ Checkout code
  ├─ Set up Docker Buildx
  ├─ Log in to registries
  ├─ Extract metadata
  └─ Build and push Docker image

✓ Verify Image - PASSED
  ├─ Pull image
  ├─ Inspect image
  └─ Verification successful
```

### Image Published To

After successful pipeline:

- **GitHub Container Registry (GHCR)**
  ```
  ghcr.io/<owner>/<repo>:latest
  ghcr.io/<owner>/<repo>:main
  ghcr.io/<owner>/<repo>:<commit-sha>
  ```

- **Docker Hub** (if configured)
  ```
  docker.io/<docker-username>/ecommerce-api:latest
  ```

## Pulling Images

### From GitHub Container Registry

```bash
docker login ghcr.io -u <your-github-username> -p <GITHUB_TOKEN>
docker pull ghcr.io/<owner>/<repo>:latest
docker run -p 8000:8000 ghcr.io/<owner>/<repo>:latest
```

### From Docker Hub

```bash
docker login
docker pull <docker-username>/ecommerce-api:latest
docker run -p 8000:8000 <docker-username>/ecommerce-api:latest
```

## Troubleshooting

### "Authentication failed" Error

**Problem**: Cannot push to Docker Hub or GHCR
- **Solution**: Verify token has read/write permissions
- **Debug**: Re-generate token with correct scopes
- **Check**: Confirm secret name matches workflow exactly

### "Image not found" When Pulling

**Problem**: Image exists but cannot pull
- **Solution**: Verify you're authenticated to the registry
- **Command**: `docker login ghcr.io` for GHCR

### Workflow Not Triggering

**Problem**: Push to main doesn't trigger workflow
- **Solution**: Verify `.github/workflows/build.yml` exists and is valid
- **Debug**: Check workflow syntax in Actions → Workflows
- **Check**: Branch name must be `main` (not `master`)

### Container Won't Start

**Problem**: `docker run` fails
- **Solution**: Add required environment variables:
  ```bash
  docker run -p 8000:8000 \
    -e DATABASE_URL="postgresql://user:pass@host/db" \
    ghcr.io/<owner>/<repo>:latest
  ```
- **Debug**: Check logs: `docker logs <container-id>`

## Advanced Configuration

### Conditional Deployments

To deploy only when tests pass, add to deploy step:

```yaml
- name: Deploy
  if: success()  # Only runs if all previous steps passed
  run: |
    # Your deployment commands
```

### Build on Schedule

Add to workflow triggers:

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
```

### Multi-Platform Builds

Docker Buildx supports multiple architectures:

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    platforms: linux/amd64,linux/arm64,linux/arm/v7
    push: true
    tags: ${{ steps.meta.outputs.tags }}
```

## Best Practices

1. ✅ Store sensitive data in Secrets, not Variables
2. ✅ Rotate tokens regularly (quarterly minimum)
3. ✅ Use specific Docker image tags in production
4. ✅ Monitor workflow runs weekly
5. ✅ Document any custom environment variables
6. ✅ Test locally before pushing to repository
7. ✅ Keep dependencies updated regularly

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Build Action](https://github.com/docker/build-push-action)
- [Docker Login Action](https://github.com/docker/login-action)
- [Container Registry Help](https://docs.github.com/en/packages/working-with-a-github-packages-registry)
