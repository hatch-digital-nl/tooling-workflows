# Deployment

## Environment Mapping

De workflows gebruiken automatisch de juiste GitHub environments op basis van de branch:

- **master/main** → `Production` environment
- **develop** → `Testing` environment
- **release/*** → `Acceptance` environment

Dit zorgt ervoor dat de juiste environment secrets en variabelen beschikbaar zijn tijdens het bouwen en deployen.

## Docker

### Basis gebruik
```yaml
name: GitFlow Docker Trigger

on:
  push:
    branches:
      - master
      - develop
      - release/*

jobs:
  deploy:
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/deploy_docker.yml@master
    permissions:
      contents: read
      packages: write
    with:
      image_name: myapp
```

### Met custom build arguments
Voor React apps of andere projecten die environment-specifieke variabelen nodig hebben tijdens de build:

```yaml
name: GitFlow Docker Trigger

on:
  push:
    branches:
      - master
      - develop
      - release/*

jobs:
  deploy:
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/deploy_docker.yml@master
    permissions:
      contents: read
      packages: write
    with:
      image_name: myapp
      build_args: |
        CUSTOM_API_URL=${{ secrets.CUSTOM_API_URL }}
        CUSTOM_ENV=${{ vars.CUSTOM_ENV }}
```

### Standaard beschikbare build arguments
De workflow geeft automatisch deze build arguments door:
- `REACT_APP_API_URL` (van secrets)
- `REACT_APP_ENV` (van vars)
- `API_BASE_URL` (van secrets)
- `NODE_ENV` (van vars)

## Docker + Kustomize
In de helm chart definieer je een Image, deze moet je meegeven als `kustomize_image`. De `image_tag` kan je vanuit de andere variant updaten.

Beide workflows gebruiken automatisch de juiste environments:
- **deploy_docker.yml**: Voor environment secrets tijdens build
- **deploy_kustomize.yml**: Voor environment secrets tijdens kustomize update

```yaml
name: GitFlow Docker + Kustomize Trigger

on:
  push:
    branches:
      - master
      - develop
      - release/*

jobs:
  deploy:
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/deploy_docker.yml@master
    permissions:
      contents: read
      packages: write
    with:
      image_name: myapp

  update-kustomize:
    needs: deploy
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/deploy_kustomize.yml@master
    with:
      kustomize_image: myapp:develop
      image_tag: ${{ needs.deploy.outputs.image_sha_tag }}
```
# Process

## GitFlow

```yaml
name: Gitflow Trigger

on:
  push:
    branches:
      - master
      - develop
      - feature/*
      - release/*
      - hotfix/*

jobs:
  call-gitflow:
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/process_gitflow.yml@master
    with:
      main_branch: master
      develop_branch: develop
```


## GitFlow

```yaml
name: Kustomize Merge Trigger

on:
  push:
    branches:
      - master
      - develop
      - release/*

jobs:
  call-kustomize:
    uses: hatch-digital-nl/tooling-workflows/.github/workflows/process_kustomize.yml@master
    permissions:
      contents: write
      pull-requests: write
    with:
      main_branch: master
      develop_branch: develop
```


