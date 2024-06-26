# About
GitHub action to build container images using [Moby Buildkit](https://github.com/moby/buildkit)'s Buildkitd daemons and buildctl commandline tool.

# Example
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
            build-args: |
               TEST=value
               TEST2=value2
            use-registry-cache: true
            cache-mode: 'max'

```

## Without using docker/metadata-action

In the example above I'm using the [docker/metadata-action](https://github.com/docker/metadata-action) to generate the tags. This can be skipped, but then the tags should be passed as a string, with each tag on a new line, for example:
```
- name: Build image
        uses: maaroen/buildkit-build-push-action@main
        with:
            platforms: 'linux/amd64'
            tags: |
              user/app:latest
              user/app:1.0.0
            buildkit-daemon-address: 'tcp://buildkitd:1234'   

```

# Customizing

## inputs

> `List` type is a newline-delimited string
> ```yaml
> tags: |
>   user/app:1.0.0
>   user/app:latest
> ```

> `CSV` type is a comma-delimited string
> ```yaml
> tags: name/app:latest,name/app:1.0.0
> ```

Following inputs can be used as `step.with` keys
| Name               | Type        | Description                                 |
|--------------------|-------------|---------------------------------------------|
| `context-folder`   | String      | Folder to use as context during image build |
| `dockerfile-folder`   | String      | Location of the Dockerfile to build the image |
| `platforms`   | List/CSV      | Platform(s) that the image should be build for, multiple platforms can be specified comma separated (linux/amd64,linux/arm64) |
| `dockerfile-name`   | String      | Name of the Dockerfile to use for the build |
| `tags`   | List      | Name of the Dockerfile to use for the build (You can use the tags output from docker/metadata-action as input  |
| `buildkit-daemon-address`   | String      | Address of the buildkit daemon to use (for example tcp://buildkitd:1234 |
| `push` | Boolean | Defines wether the image should be pushed to the registry or not, default is true |
| `build-args` | List | The build arguments that are required for building the image |
| `use-registry-cache` | Boolean | Wether to cache layers in a registry for faster subsequent builds (Default true) |
| `cache-mode` | String | Wether to use min or max mode for caching (max caches all layers, min only the resulting image) |
