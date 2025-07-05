# Container Registry Cleanup Action

A GitHub Action to automatically cleanup old container images from GitHub Container Registry (ghcr.io).

## Features

- 🗑️ Delete old container images based on age
- 🛡️ Protect important tags (latest, main, master, etc.)
- 📊 Keep minimum number of versions
- ⚡ Configurable batch processing
- 🔒 Safe deletion with detailed logging
- 🎯 Support for both tagged and untagged versions

## Usage

### Basic Example

```yaml
name: Cleanup Container Registry

on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
    - name: Cleanup old container images
      uses: ruedigerp/container-registry-cleanup@v1
      with:
        package-name: 'my-app'
        token: ${{ secrets.PAT_TOKEN }}
        days-old: 21
        min-versions-to-keep: 3
```

### Advanced Example

```yaml
name: Advanced Container Cleanup

on:
  workflow_dispatch:

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
    - name: Cleanup multiple packages
      uses: ruedigerp/container-registry-cleanup@v1
      with:
        package-name: 'my-app'
        token: ${{ secrets.PAT_TOKEN }}
        days-old: 14
        min-versions-to-keep: 5
        max-versions-per-run: 20
        protected-tags: 'latest|stable|v[0-9]+\\.[0-9]+\\.[0-9]+'
        delete-untagged-only: false
```

### Multiple Packages

```yaml
jobs:
  cleanup:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package: ['frontend', 'backend', 'worker']
    steps:
    - name: Cleanup ${{ matrix.package }}
      uses: ruedigerp/container-registry-cleanup@v1
      with:
        package-name: ${{ matrix.package }}
        token: ${{ secrets.PAT_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `package-name` | Name of the container package to cleanup | Yes | - |
| `token` | GitHub token with `packages:write` permission | Yes | - |
| `days-old` | Delete versions older than this many days | No | `21` |
| `min-versions-to-keep` | Minimum number of old versions to keep | No | `3` |
| `max-versions-per-run` | Maximum versions to delete per run | No | `10` |
| `protected-tags` | Regex pattern for protected tags (pipe-separated) | No | `latest\|main\|master\|develop\|dev` |
| `delete-untagged-only` | Only delete untagged versions | No | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `deleted-count` | Number of versions deleted |
| `total-versions` | Total versions found |
| `old-versions` | Number of old versions found |

## Token Setup

1. Create a Personal Access Token (PAT):
   - Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Select scopes: `read:packages`, `write:packages`, `delete:packages`

2. Add the token as a repository secret:
   - Repository → Settings → Secrets and variables → Actions
   - Name: `PAT_TOKEN`
   - Value: Your created token

## Protected Tags

By default, these tags are protected and won't be deleted:
- `latest`
- `main`
- `master` 
- `develop`
- `dev`

You can customize this with the `protected-tags` input using regex patterns.

## Safety Features

- **Minimum versions**: Always keeps a minimum number of old versions
- **Batch processing**: Limits deletions per run to avoid overwhelming the API
- **Protected tags**: Prevents deletion of important tags
- **Detailed logging**: Shows exactly what's being deleted and why
- **Dry-run capability**: Set `max-versions-per-run: 0` to see what would be deleted

## Examples by Use Case

### Conservative Cleanup (Untagged Only)
```yaml
- uses: ruedigerp/container-registry-cleanup@v1
  with:
    package-name: 'my-app'
    token: ${{ secrets.PAT_TOKEN }}
    delete-untagged-only: true
```

### Aggressive Cleanup (Keep Only Latest Releases)
```yaml
- uses: ruedigerp/container-registry-cleanup@v1
  with:
    package-name: 'my-app'
    token: ${{ secrets.PAT_TOKEN }}
    days-old: 7
    min-versions-to-keep: 1
    protected-tags: 'latest|v[0-9]+\\.[0-9]+\\.[0-9]+'
```

### Large Repository Cleanup
```yaml
- uses: ruedigerp/container-registry-cleanup@v1
  with:
    package-name: 'my-app'
    token: ${{ secrets.PAT_TOKEN }}
    max-versions-per-run: 50
    days-old: 30
```

## Troubleshooting

### Common Issues

1. **403 Forbidden**: Check that your PAT has the correct permissions
2. **Package not found**: Ensure the package name is correct and accessible
3. **No versions deleted**: Check that versions exist and meet the age criteria

### Debug Mode

Add this step before the cleanup to debug:

```yaml
- name: Debug package info
  run: |
    gh api /user/packages/container/my-app | jq
    gh api /user/packages/container/my-app/versions | jq '.[0:3]'
  env:
    GITHUB_TOKEN: ${{ secrets.PAT_TOKEN }}
```

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Contributing

Contributions welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.