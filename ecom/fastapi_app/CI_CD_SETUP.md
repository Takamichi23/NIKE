# CI/CD Pipeline Setup Documentation

## Overview
This document describes the CI/CD pipeline for the FastAPI E-commerce API application using Docker and GitHub Actions.

## Architecture

The CI/CD pipeline consists of three main jobs:

### 1. **Lint and Test** (`lint-and-test`)
- **Trigger**: On push to `main` or pull requests
- **Environment**: Ubuntu latest with Python 3.12
- **Steps**:
  - Checkout code
  - Set up Python environment with pip caching
  - Install dependencies from `requirements.txt`
  - Run linting checks (pylint)
  - Run type checking (pyright)
  - Execute unit tests with coverage reporting (pytest)

**Key Features**:
- `continue-on-error: true` allows the pipeline to proceed even if linting fails
- Cached pip dependencies for faster builds
- Coverage reports generated for monitoring code quality

### 2. **Build and Push** (`build-and-push`)
- **Trigger**: After successful lint-and-test job
- **Environment**: Ubuntu latest with Docker Buildx
- **Steps**:
  - Checkout code
  - Set up Docker Buildx for multi-platform builds
  - Authenticate with GitHub Container Registry (GHCR)
  - Authenticate with Docker Hub (if credentials provided)
  - Extract metadata (tags, labels, version info)
  - Build and push Docker image to registries

**Docker Image Tagging Strategy**:
```
ghcr.io/<owner>/<repo>:latest
ghcr.io/<owner>/<repo>:<branch-name>
ghcr.io/<owner>/<repo>:<git-sha>
docker.io/<docker-username>/ecommerce-api:latest
```

**Caching**:
- Uses GitHub Actions cache (type=gha) for Docker layer caching
- Significantly reduces build time on subsequent runs

### 3. **Verify Image** (`verify-image`)
- **Trigger**: After successful build-and-push job
- **Environment**: Ubuntu latest with Docker CLI
- **Steps**:
  - Checkout code
  - Authenticate with GHCR
  - Pull the built image
  - Inspect image metadata
  - Verify image integrity

## Environment Variables & Secrets Required

### GitHub Secrets
These must be configured in your repository settings:
- `GITHUB_TOKEN`: Automatically provided by GitHub Actions (used for GHCR)
- `DOCKERHUB_TOKEN`: Docker Hub personal access token (optional, for Docker Hub push)

### GitHub Variables
These should be set in repository settings:
- `DOCKER_USERNAME`: Your Docker Hub username (optional)

## Local Development

### Build Docker Image Locally
```bash
docker build -t ecommerce-api:latest -f Dockerfile .
```

### Run Container Locally
```bash
docker run -p 8000:8000 \
  -e DATABASE_URL="your-database-url" \
  ecommerce-api:latest
```

### Using Docker Compose
```bash
docker-compose up -d
```

## Deployment

The built Docker image can be deployed to any container orchestration platform:

### Docker Swarm
```bash
docker service create \
  --name ecommerce-api \
  --publish 80:8000 \
  -e DATABASE_URL="your-database-url" \
  ghcr.io/<owner>/<repo>:latest
```

### Kubernetes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ecommerce-api
spec:
  containers:
  - name: api
    image: ghcr.io/<owner>/<repo>:latest
    ports:
    - containerPort: 8000
    env:
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: api-secrets
          key: database-url
    livenessProbe:
      httpGet:
        path: /health
        port: 8000
      initialDelaySeconds: 10
      periodSeconds: 30
```

## Dockerfile Best Practices

The Dockerfile uses a **multi-stage build** approach:

1. **Builder Stage**: Compiles dependencies
   - Uses `python:3.12-slim` base image
   - Installs build tools and dependencies
   - Creates layered cache for faster rebuilds

2. **Runtime Stage**: Runs the application
   - Minimal final image size
   - Non-root user (`appuser`) for security
   - Health check endpoint: `/health`
   - Environment variables configuration

## Health Check

The application includes a health check endpoint:

```
GET /health
Response: {"status": "ok"}
```

Docker checks this every 30 seconds with:
- 5-second timeout
- 5-second startup period
- 3 retries before marking unhealthy

## Troubleshooting

### Pipeline Failures

**Issue**: Build fails during dependency installation
- **Solution**: Check `requirements.txt` for compatibility with Python 3.12
- **Debug**: Run `pip install -r requirements.txt` locally

**Issue**: Docker image push fails
- **Solution**: Verify Docker Hub token has appropriate permissions
- **Debug**: Check repository secrets are correctly configured

**Issue**: Tests fail in CI but pass locally
- **Solution**: Ensure DATABASE_URL is properly configured in test environment
- **Debug**: Add debug logging to identify environment differences

### Image Issues

**Issue**: Large image size
- **Solution**: Use multi-stage builds (already implemented)
- **Solution**: Add unnecessary files to `.dockerignore`

**Issue**: Container won't start
- **Solution**: Check logs: `docker logs <container-id>`
- **Debug**: Verify all environment variables are set

## Performance Optimization

1. **Layer Caching**: Dockerfile orders dependencies by change frequency
2. **GitHub Actions Cache**: Reduces rebuild time by 60-70%
3. **Buildx Cache**: Stores layers in local cache storage
4. **Slim Base Image**: Uses `python:3.12-slim` instead of full image

## Security Considerations

1. **Non-root User**: Application runs as `appuser`, not `root`
2. **Read-only Filesystem**: Consider `--read-only` flag for production
3. **Resource Limits**: Set CPU/memory limits in orchestration platform
4. **Secret Management**: Never commit secrets to repository
5. **Image Scanning**: Use `docker scan` or registry scanning for vulnerabilities

## Monitoring & Logging

The application exposes:
- Health check endpoint at `/health`
- Standard output logs from FastAPI/Uvicorn
- Container metrics available through Docker API

## Next Steps

1. Configure secrets in GitHub repository settings
2. Push changes to trigger first CI/CD run
3. Monitor build logs in Actions tab
4. Deploy image to your container platform
5. Set up monitoring and alerting
