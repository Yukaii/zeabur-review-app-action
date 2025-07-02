# Zeabur Review App Action

A standalone GitHub composite action for deploying, managing, and cleaning up Zeabur review apps for pull requests with commit-level isolation.

> **📦 Standalone Action**: This action is now available as a standalone repository and can be used in any project without needing to copy files.

## Features

- 🚀 **Automated PR review app deployment** with commit-specific isolation
- 🧹 **Automatic cleanup** when PRs are closed
- 🏷️ **Commit-specific image tagging** for Docker images
- 🔧 **Configurable service management** (ignore, cleanup, update patterns)
- 📝 **Status checking** for active review apps
- 🎯 **Project-independent** configuration

## Usage

### As a GitHub Action

```yaml
name: Deploy Review App
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Deploy Review App
        uses: Yukaii/zeabur-review-app-action@main
        with:
          action: deploy
          zeabur-api-key: ${{ secrets.ZEABUR_API_KEY }}
          zeabur-project-id: ${{ secrets.ZEABUR_PROJECT_ID }}
          pr-number: ${{ github.event.number }}
          commit-sha: ${{ github.sha }}
          project-name: "My Project"
          domain-prefix: "myapp"
          template-file: "zeabur.yaml"
```

### As a Standalone Script

Install via npm (when published):
```bash
npm install -g zeabur-review-app
```

Or use directly:
```bash
# Deploy a review app
PR_NUMBER=123 COMMIT_SHA=abc1234 ./zeabur-review-app.sh deploy

# Clean up review app services
PR_NUMBER=123 ./zeabur-review-app.sh cleanup

# Check status
PR_NUMBER=123 ./zeabur-review-app.sh status
```

## Configuration

### Environment Variables

**Required:**
- `ZEABUR_API_KEY` - Zeabur API token
- `ZEABUR_PROJECT_ID` - Target Zeabur project ID
- `PR_NUMBER` - Pull request number

**Optional:**
- `COMMIT_SHA` - Git commit hash (auto-detected if not provided)
- `PROJECT_NAME` - Project name for review apps (default: "Review App")
- `IGNORED_SERVICES` - Comma-separated service names to exclude
- `CLEANUP_SERVICES` - Comma-separated service names to cleanup after deployment
- `UPDATE_IMAGE_SERVICES` - Comma-separated service patterns to update with commit tags
- `DOMAIN_PREFIX` - Domain prefix for review apps (default: "app")
- `IMAGE_TAG_PREFIX` - Image tag prefix (default: "sha")
- `ZEABUR_TEMPLATE_FILE` - Path to zeabur.yaml template (default: "./zeabur.yaml")
- `ZEABUR_CONFIG_FILE` - Path to config file (default: "./zeabur-config.env")

### Configuration File

Create a `zeabur-config.env` file in your project root:

```bash
# Project-specific settings
PROJECT_NAME="My Project"
IGNORED_SERVICES="Worker Service"
CLEANUP_SERVICES="Database"
UPDATE_IMAGE_SERVICES="Backend,Frontend"
DOMAIN_PREFIX="myapp"
IMAGE_TAG_PREFIX="sha"
```

### Template File

The script requires a `zeabur.yaml` template file that defines your services. The script will:

1. Remove services listed in `IGNORED_SERVICES`
2. Add PR and commit suffixes to all service names
3. Update service dependencies to match new names
4. Update Docker image tags for services matching `UPDATE_IMAGE_SERVICES`
5. Set custom domain variables

Example template structure:
```yaml
apiVersion: zeabur.com/v1
kind: Template
metadata:
  name: My Project
spec:
  description: My project description
  services:
    - name: Database
      template: PREBUILT_V2
      spec:
        source:
          image: postgres:14
    - name: Backend
      template: PREBUILT_V2
      dependencies:
        - Database
      spec:
        source:
          image: myorg/backend:latest
```

## Action Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `action` | Action to perform (deploy/cleanup/status) | Yes | `deploy` |
| `zeabur-api-key` | Zeabur API key | Yes | - |
| `zeabur-project-id` | Zeabur project ID | Yes | - |
| `pr-number` | Pull request number | Yes | - |
| `commit-sha` | Git commit SHA | No | Auto-detected |
| `project-name` | Project name for review apps | No | `Review App` |
| `ignored-services` | Services to exclude from review apps | No | `""` |
| `cleanup-services` | Services to cleanup after deployment | No | `""` |
| `update-image-services` | Service patterns to update with commit tags | No | `""` |
| `domain-prefix` | Domain prefix for review apps | No | `app` |
| `image-tag-prefix` | Image tag prefix | No | `sha` |
| `template-file` | Path to zeabur.yaml template | No | `zeabur.yaml` |
| `config-file` | Path to config file | No | `zeabur-config.env` |

## Action Outputs

| Output | Description |
|--------|-------------|
| `review-app-url` | URL of the deployed review app |
| `review-app-project-name` | Name of the deployed project |
| `review-app-project-id` | ID of the deployed project |
| `review-app-region` | Region where the app is deployed |
| `review-app-domain` | Domain name of the review app |

## How It Works

1. **Service Isolation**: Each PR gets unique service names with the pattern `{service-name}-pr-{number}-{commit}`
2. **Image Tagging**: Services matching `UPDATE_IMAGE_SERVICES` get commit-specific image tags
3. **Domain Management**: Each deployment gets a unique domain like `{prefix}-pr-{number}-{commit}.zeabur.app`
4. **Dependency Updates**: Service dependencies are automatically updated to match new names
5. **Cleanup**: Services can be automatically cleaned up after deployment or when PRs close

## Requirements

- `curl` - For API requests
- `jq` - For JSON processing
- `yq` - For YAML processing
- `git` - For commit hash detection (optional)

## Examples

### Basic Review App Deployment

```yaml
- name: Deploy Review App
  uses: Yukaii/zeabur-review-app-action@main
  with:
    action: deploy
    zeabur-api-key: ${{ secrets.ZEABUR_API_KEY }}
    zeabur-project-id: ${{ secrets.ZEABUR_PROJECT_ID }}
    pr-number: ${{ github.event.number }}
```

### Advanced Configuration

```yaml
- name: Deploy Review App
  uses: Yukaii/zeabur-review-app-action@main
  with:
    action: deploy
    zeabur-api-key: ${{ secrets.ZEABUR_API_KEY }}
    zeabur-project-id: ${{ secrets.ZEABUR_PROJECT_ID }}
    pr-number: ${{ github.event.number }}
    commit-sha: ${{ github.sha }}
    project-name: "Disfactory"
    ignored-services: "Worker,Cache"
    cleanup-services: "Database"
    update-image-services: "Backend,Frontend"
    domain-prefix: "disfactory"
    template-file: "deployment/zeabur.yaml"
```

### Cleanup on PR Close

```yaml
name: Cleanup Review App
on:
  pull_request:
    types: [closed]

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Cleanup Review App
        uses: Yukaii/zeabur-review-app-action@main
        with:
          action: cleanup
          zeabur-api-key: ${{ secrets.ZEABUR_API_KEY }}
          zeabur-project-id: ${{ secrets.ZEABUR_PROJECT_ID }}
          pr-number: ${{ github.event.number }}
```

## License

MIT

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test your changes
4. Submit a pull request

## Support

For issues and questions, please open an issue in the repository.