# docker-deploy-action

Action to deploy docker container

```yml
name: Release

on:
  release:
    types:
      - released
  workflow_dispatch:

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.prep.outputs.version }}
    steps:
      - name: Prepare
        id: prep
        shell: bash
        run: |
          DOCKER_IMAGE=${{ secrets.DOCKER_REGISTRY_URL }}/${{ secrets.DOCKER_IMAGE_NAME }}
          VERSION=${GITHUB_REF#refs/tags/}
          TAGS="${DOCKER_IMAGE}:${VERSION}"
          if [[ $VERSION =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
              MINOR=${VERSION%.*}
              MAJOR=${MINOR%.*}
              TAGS="$TAGS,${DOCKER_IMAGE}:${MINOR},${DOCKER_IMAGE}:${MAJOR},${DOCKER_IMAGE}:latest"
          fi
          echo ::set-output name=tags::${TAGS}
          echo ::set-output name=version::${VERSION}
          echo ::set-output name=created::$(date -u +'%Y-%m-%dT%H:%M:%SZ')
        # build docker image
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [build]
    steps:
      - name: Deploy service
        uses: sitkoru/docker-deploy-action@v1
        with:
          name: ${{ secrets.SERVICE_NAME }}
          version: ${{ needs.build.outputs.version }}
          host: ${{ secrets.DEPLOY_HOST }}
          user: ${{ secrets.DEPLOY_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          dir: ${{ secrets.HOST_WORKING_DIR }}
          script: ${{ secrets.HOST_COMPOSE_SCRIPT }}
          # uncomment if service needs Vault
          # vault_policy: ${{ secrets.VAULT_POLICY }}
          # vault_container: ${{ secrets.VAULT_CONTAINER }}
          # vault_username: ${{ secrets.VAULT_USERNAME }}
          # vault_password: ${{ secrets.VAULT_PASSWORD }}
          # vault_period: ${{ secrets.VAULT_PERIOD }}
```
