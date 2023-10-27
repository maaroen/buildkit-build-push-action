# About
GitHub action to build container images using [Moby Buildkit](https://github.com/moby/buildkit)'s Buildkitd daemons and buildctl commandline tool.

# Usage

This task can be used on it's own, or in combination with [docker/metadata-action](https://github.com/docker/metadata-action)

## Using docker/metadata-action
You could use this step to generate the tags:
```
env: 
      DOCKER_CONFIG: "~/.docker/"
      IMAGE_REPOSITORY: git.nederlof.dev/maaroen/minecraft-servers/vault-hunters/third-edition
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
```
After this step with id `meta` you can access the tags like this: `${{ steps.meta.outputs.tags }}` and pass them to this buildkit action.

## Without using docker/metadata-action
The expected format

In the example below I'm also using the [docker/metadata-action](https://github.com/docker/metadata-action) to generate the tags. This can be skipped, but then the tags should be passed as a string, with each tag on a new line, for example:
```
user/app:latest
user/app:1.0.0
```
