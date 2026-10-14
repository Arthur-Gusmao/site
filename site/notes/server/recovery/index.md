# Recovery

[Server guide](/notes/server/)

## Restore a release

    doas werc rollback
    doas werc status

Rollback pauses automatic updates. If there is no previous container,
deploy a known full commit SHA instead:

    doas werc update FULL_COMMIT_SHA

Resume when ready:

    doas werc resume

## Interrupted update

The deployment script restores `werc-site.previous` when the live container
is absent, then retries deployment. The OpenRC service also recovers this
rename when starting after boot.

## Rebuild the host

Follow [installation](/notes/server/installation/), restore the private tunnel
token, and deploy a known commit. Git and GHCR contain the public site;
the tunnel token must come from your private backup.
