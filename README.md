# Portainer Stack Deploy

GitHub Action to push a compose file to an existing Portainer stack **and
actually force a redeploy** — prune + re-pull the image, even when the image
reference in the compose file is an unchanged `:latest` tag.

## Why not one of the existing marketplace actions?

Several Portainer deploy actions accept `pull-image`/`prune-stack`-style
inputs but never send them to Portainer's `PUT /api/stacks/{id}` call. When
the compose text doesn't change between deploys (a plain `image: repo:latest`
never changes as a string), Docker Swarm has no diff to react to and skips
the pull — the CI step reports success, but the running container silently
stays on the old image. This action sends `prune` and
`repullImageAndRedeploy` explicitly, which is what actually makes Portainer
force the redeploy.

It's a plain bash **composite action** (no `npm run build` / bundled
`dist/index.js` to go stale) — the whole implementation is in
[`action.yml`](./action.yml).

## Usage

```yaml
- name: Deploy on Portainer
  uses: LucasSantos96/portainer-stack-deploy@v1
  with:
    portainer-host: "https://portainer.example.com"
    username: "admin"
    password: ${{ secrets.PORTAINER_PASSWORD }}
    stack-name: "my_stack"
    stack-definition: "docker/docker-compose.portainer.yml"
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `portainer-host` | yes | — | Portainer base URL, e.g. `https://portainer.example.com` |
| `username` | yes | — | Portainer username (use a CI-specific account) |
| `password` | yes | — | Portainer password |
| `stack-name` | yes | — | Name of the existing stack to update |
| `stack-definition` | yes | — | Path to the compose file to push |
| `endpoint-id` | no | stack's current endpoint | Override the target environment/endpoint ID |
| `prune` | no | `true` | Remove services no longer present in the new definition |
| `repull` | no | `true` | Force a re-pull + redeploy even if the compose text is unchanged |

## Notes

- The stack must already exist in Portainer (created once via the UI or
  another tool) — this action only updates it, it does not create new
  stacks.
- Existing stack environment variables (set in the Portainer UI) are
  preserved automatically; this action only replaces the compose file
  content.
- Requires `curl` and `jq`, both preinstalled on `ubuntu-latest` GitHub
  runners.
