# Github actions
Shared Github Actions Workflows

Most jobs expect a Personal Access Token to be set as an organizational secret named GH_ORG_TOKEN which has permissions for:

repo
write:packages
You can generate this token here: https://github.com/settings/tokens/new?scopes=repo,write:packages

## Example of usage

### Build -> Release -> Deploy workflow

```yaml
name: CI
on:
  push:
    branches: [ "main" ]

jobs:
  mvn:
    uses: property-xyz/actions/.github/workflows/build-quarkus-jvm-17.yaml@main
    secrets: inherit
    with:
      isNative: false
  docker-helm-release:
    needs: mvn
    uses: property-xyz/actions/.github/workflows/release-helm.yaml@main
    secrets: inherit
    with:
      registry: ghcr.io
      version: ${{ needs.mvn.outputs.version }}
      tag: ${{ needs.mvn.outputs.tag }}
  argocd:
    needs:
      - mvn
      - docker-helm-release
    uses: property-xyz/actions/.github/workflows/deploy-chart.yaml@main
    secrets: inherit
    with:
      version: ${{ needs.mvn.outputs.version }}
```