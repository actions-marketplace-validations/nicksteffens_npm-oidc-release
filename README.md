# NPM OIDC Release

GitHub Action for publishing to NPM using OIDC authentication with release-it.

## Prerequisites

1. Configure NPM Trusted Publishing on npmjs.com:
   - Go to package settings → "Configure Trusted Publishing"
   - Set repository and workflow path

2. Your repo needs:
   - `release-it` as a dev dependency
   - `.release-it.json` with `"npm": { "skipChecks": true }`
   - Node version in `package.json` (volta or engines)

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
