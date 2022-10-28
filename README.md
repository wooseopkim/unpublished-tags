# Unpublished Tags Action

## Summary

This is a GitHub action to fetch tags that are in the GitHub repository but not published on the Docker Hub repository yet.

## User Guide

Before you use this action, you'll need to call `actions/checkout` with the repository of interest. Note that you need to provide input `fetch-depth: 0` to the checkout action.

### Inputs

|NAME|REQUIRED|EXAMPLE|
|-|-|-|
|dockerhub-repository|true|`org/repository`|
|first-tag|false (default: `<first tag>`)|`v1.2.3`|
|checkout-path|false (default: `.`)|`dir`|

### Outputs

|NAME|EXAMPLE|NOTE|
|-|-|-|
|tags|`["1.23.1", "1.23.0", ..., "0.1.0"]`|This is the list of Docker tags, not git tags.|

### Example

```yaml
jobs:
  tags:
    runs-on: ubuntu-latest
    outputs:
      json: ${{ steps.unpublished.outputs.json }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - name: Get unpublished tags
        id: unpublished
        uses: wooseopkim/unpublished-tags@v2
        with:
          dockerhub-repository: org/repository
  build:
    needs: [tags]
    runs-on: ubuntu-latest
    strategy:
      max-parallel: 1
      matrix:
        tag: ${{ fromJSON(needs.tags.outputs.json) }}
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build
        run: ./build.sh --tag=${{ matrix.tag }}
```

## Limitations and caution

This action is made for personal use and functions are limited. For example, you can only fetch published tags from Docker Hub, not from other registries such as GitHub Container Registry. PRs are welcome though.

You can only get the tags that actually belong to branches on GitHub. The tags which does not belong to a branch will be discarded. Thus the output might not include tags you can see on GitHub or in the `git tag -l` result. Use `git tag --merged <ref>` instead.

This action assumes that:

- You only published tagged commits,
- The git tag names are either `<Docker tag name>` or `v<Docker tag name>`,
- If a tag is published, all previous tags are already published.
