---
tags:
  - cheatsheet
  - git
---
# Github Actions Notes

This document includes a collection of notes on Github Actions that I have found
challenging to remember. For additional information, consulting the official
documentation is recommended

## Examples:

**GitHub Actions workflow for building and pushing a Docker image to a private
registry**

```yaml
name: Build and push a Docker image on a private registry

on:
  push:
    # Publish `main` as Docker `latest` image
    branches:
      - main
    # Publish `v1.2.3` tags as releases
    tags:
      - v*
  # Run some test for PRs to the main branch
  pull_request:
    branches:
      - main

env:
  IMAGE_NAME: foobar
  REGISTRY: registry.example.com
  PROJECT: infra

jobs:
  # For example this job ckecks the Go formatting
  gofmt:
    name: Checkout code and set up Go version
    runs-on: some-private-agent
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Go
      uses: actions/setup-go@v3
      with:
        go-version: '1.22' # Specify the Go version

    - name: Check code formatting
      run: |
        unformatted=$(gofmt -l .)
        if [ -n "$unformatted" ]; then
          echo "Unformatted files detected:"
          echo "$unformatted"
          exit 1
        fi

  push:
    # Ensure gofmt job passes before pushing image.
    needs: gofmt
    runs-on: some-private-agent
    if: github.event_name == 'push'

    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build . --file Dockerfile --tag $IMAGE_NAME

      - name: Set DOCKER_CONFIG environment variable
        run: |
          echo "DOCKER_CONFIG=$RUNNER_TEMP/.docker" >> $GITHUB_ENV
          mkdir -p $RUNNER_TEMP/.docker

      - name: Log into registry.example.com
        run: echo "${{ secrets.DOCKER_PASS }}" | docker login $REGISTRY -u admin --password-stdin

      - name: Push image
        run: |
          IMAGE_ID=$REGISTRY/$PROJECT/$IMAGE_NAME

          # Strip git ref prefix from version
          VERSION=$(echo "${{ github.ref }}" | sed -e 's,.*/\(.*\),\1,')

          # Strip "v" prefix from tag name
          [[ "${{ github.ref }}" == "refs/tags/"* ]] && VERSION=$(echo $VERSION | sed -e 's/^v//')

          # Use Docker `latest` tag convention
          [ "$VERSION" == "main" ] && VERSION=latest

          # verbose
          echo IMAGE_ID=$IMAGE_ID
          echo VERSION=$VERSION

          # tag the built image and push it to Docker Hub
          docker tag $IMAGE_NAME $IMAGE_ID:$VERSION
          docker push $IMAGE_ID:$VERSION

      - name: Logout from registry.example.com
        run: docker logout $REGISTRY

      - name: Remove DOCKER_CONFIG environment variable
        run: rm -rf $RUNNER_TEMP/.docker
```
