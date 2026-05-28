# Publishing the Docker image

The mock is published to Docker Hub as
[`eerootsus/govsso-mock`](https://hub.docker.com/r/eerootsus/govsso-mock).
The repository has **mutable tags disabled**, so every push must use a unique
tag — `latest` is not used. Consumers (KMT local-dev compose, the dev cluster's
mirror) pin the immutable `sha-<short>` tag of the image they want.

Publishing is currently **manual** — there is no CI workflow in this repo.

## Prerequisites

- A Docker Hub access token with `read & write` on `eerootsus/govsso-mock`
  (Docker Hub → *Settings → Personal Access Tokens*).
- `docker login` once so the daemon has the token cached:

  ```sh
  docker login -u eerootsus
  # paste the access token (NOT the account password) when prompted
  ```

  (Verify: `docker info | grep Username` prints `eerootsus`.)

## Standard publish (any master commit)

Run from the repo root, on a clean working tree at the commit you want to
publish:

```sh
SHA=$(git rev-parse --short HEAD)
TAG="eerootsus/govsso-mock:sha-${SHA}"

docker build -f build/Dockerfile -t "$TAG" .
docker push "$TAG"

echo "Published: $TAG"
```

Then update the downstream pin (e.g. KMT's `docker-compose.yml`
`govsso-mock.image`) to the new `sha-…` tag and open an MR there.

## Notes

- **Don't push `:latest`** — the Hub repo's mutable-tag protection rejects it,
  and pinning by `sha-<short>` is what downstream consumers rely on.
- **Don't reuse a tag** — the same applies to `sha-<short>` if you ever
  rebuilt the same commit; pushes to an existing tag are rejected. Bump the
  source commit first.
- Image is `linux/amd64` only. If a different arch is needed,
  `docker buildx build --platform linux/amd64,linux/arm64 …` plus a manifest
  push works, but we have no consumers for arm64 yet.
