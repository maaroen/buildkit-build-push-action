# About
GitHub action to build container images using [Moby Buildkit](https://github.com/moby/buildkit)'s Buildkitd daemons and buildctl commandline tool.

# Usage

This task can be used on it's own, or in combination with [docker/metadata-action](https://github.com/docker/metadata-action)

## Example
```
name: Gitea Actions
run-name: Build image🚀
on: [push]

jobs:
  push_to_registry:
    runs-on: ubuntu-latest
    container: 
      image: maaroen/act-runner-buildkit-kubectl
    env: 
      DOCKER_CONFIG: "~/.docker/"
      IMAGE_REPOSITORY: registry/imagename
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0 # all history for all branches and tags                
      - name: Generate Docker metadata
        id: meta
        uses: docker/metadata-action@v3
        with:
          images: |
            ${{ env.IMAGE_REPOSITORY }}                        
          tags: |
            type=semver,pattern={{version}}                        
          flavor: |
            latest=true                        
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      - name: Build image
        uses: maaroen/buildkit-build-push-action@main
        with:
            platforms: 'linux/amd64'
            tags: ${{ steps.meta.outputs.tags }}
            buildkit-daemon-address: 'tcp://buildkitd:1234'   
```

## Without using docker/metadata-action
The expected format

In the example below I'm also using the [docker/metadata-action](https://github.com/docker/metadata-action) to generate the tags. This can be skipped, but then the tags should be passed as a string, with each tag on a new line, for example:
```
user/app:latest
user/app:1.0.0
```
