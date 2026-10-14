# CI/CD

[Server guide](/notes/server/)

Pull requests build the container and test its configuration, HTTP routes,
revision marker and private-path protection. They do not publish images.

Changes to `master` run the same checks and publish
`ghcr.io/arthur-gusmao/site:COMMIT_SHA`. The host checks `master` each minute.
If its revision changed, it pulls and tests that image on port `18086`,
then replaces the container on `8080`. A failed replacement restores the old one.

    doas werc update
    doas werc update FULL_COMMIT_SHA

Only the website image updates automatically. Changes in `ops/` require
rerunning the installer from reviewed source. Update logs go to the system
log under `werc-deploy`.
