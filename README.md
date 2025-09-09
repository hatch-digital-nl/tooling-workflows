# Deployment

## Docker
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
    uses: unframed/tooling-workflows/.github/workflows/deploy_docker.yml@master
    with:
      image_name: myapp
```

## Docker + Kustomize
In de helm chart definieer je een Image, deze moet je meegeven als `kustomize_image`. De `image_tag` kan je vanuit de andere variant updaten.
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
    uses: unframed/tooling-workflows/.github/workflows/deploy_docker.yml@master
    with:
      image_name: myapp

  update-kustomize:
    needs: deploy
    uses: unframed/tooling-workflows/.github/workflows/deploy_kustomize.yml@master
    with:
      kustomize_image: ./deploy/overlays/dev
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
    uses: unframed/tooling-workflows/.github/workflows/process_gitflow.yml@master
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
    uses: unframed/tooling-workflows/.github/workflows/process_kustomize.yml@master
    with:
      main_branch: master
      develop_branch: develop
```

