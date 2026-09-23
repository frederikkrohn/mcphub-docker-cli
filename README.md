# MCPHub with Docker CLI

This image extends `samanhappy/mcphub:latest` with `docker-ce-cli` only. It
does **not** include Docker Engine, Rust, or Playwright.

The scheduled GitHub Actions workflow rebuilds the image daily with
`pull: true`; it therefore picks up the then-current upstream MCPHub `latest`
image. Every build publishes:

```text
ghcr.io/frederikkrohn/mcphub-docker-cli:latest
```

## Coolify runtime configuration

Replace the MCPHub image with:

```text
ghcr.io/frederikkrohn/mcphub-docker-cli:latest
```

Add this bind mount in addition to the existing persistent data volumes:

```text
/var/run/docker.sock:/var/run/docker.sock
```

The socket is required because the image contains only the Docker *client*;
the existing Docker daemon on the host creates and runs the stdio MCP
containers.

## Automatic Coolify deployment

1. In Coolify, enable API access and create an API token with only the
   `deploy` permission.
2. Open the MCPHub application, then **Configuration -> Webhooks**, and copy
   its authenticated **Deploy Webhook** URL.
3. In this GitHub repository, add Actions secrets:
   - `COOLIFY_DEPLOY_WEBHOOK`: the copied URL
   - `COOLIFY_TOKEN`: the deploy-only API token
4. Add the Actions variable `COOLIFY_DEPLOY_ENABLED` with value `true`.

After a successful image build, the workflow calls the deploy webhook. Until
the variable is set, it builds and publishes the image but does not deploy it.

## Security

Docker socket access grants the hub broad control over the host Docker daemon.
Only configure reviewed stdio MCP images and do not expose the socket to other
containers.
