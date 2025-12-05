# NPM OIDC Release

GitHub Action for publishing to NPM using OIDC authentication with release-it.

## Requirements

### 1. Install release-it

```bash
npm install --save-dev release-it @release-it/conventional-changelog
```

### 2. Configure release-it

Create or update `.release-it.json` with `skipChecks` enabled (required for OIDC since there's no token to validate before publishing):

```json
{
  "npm": {
    "skipChecks": true
  },
  "git": {
    "commitMessage": "chore: release v${version}"
  },
  "github": {
    "release": true
  },
  "plugins": {
    "@release-it/conventional-changelog": {
      "infile": "CHANGELOG.md",
      "preset": "conventionalcommits"
    }
  }
}
```

### 3. Configure NPM Trusted Publishing

On npmjs.com:
1. Go to your package → Settings → "Configure Trusted Publishing"
2. Set:
   - **Repository**: `your-org/your-repo`
   - **Workflow**: `.github/workflows/release.yml` (or your workflow path)
   - **Environment**: leave blank

## Usage

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      release_type:
        description: 'Release type'
        required: true
        type: choice
        default: 'release'
        options:
          - release
          - prerelease
      increment:
        description: 'Version increment (optional)'
        required: false
        default: ''
      dist_tag:
        description: 'Pre-release dist tag'
        required: false
        default: 'alpha'

permissions:
  contents: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: nicksteffens/npm-oidc-release@v1
        with:
          release_type: ${{ inputs.release_type }}
          increment: ${{ inputs.increment }}
          dist_tag: ${{ inputs.dist_tag }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `release_type` | `release` or `prerelease` | `release` |
| `increment` | Version bump: `major`, `minor`, `patch`, or blank for auto | `''` |
| `dist_tag` | NPM dist tag for prereleases | `alpha` |

## How it works

- Detects package manager (yarn or npm) from lock file
- Uses Node 22 with npm 10+ for OIDC support
- Configures git with the GitHub actor
- Installs dependencies (`yarn install --immutable` or `npm ci`)
- Runs release-it with the appropriate flags
