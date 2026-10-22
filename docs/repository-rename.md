# Repository rename migration

`icepick-offsec` is becoming **icepick-backend**; its agent consumer,
`icepick-offsec-autonomous`, is becoming **icepick-agent**. Rename the existing
GitHub repositories to preserve their history, releases, issues and PRs. This
preparation PR must remain unmerged until both destination URLs resolve. The
[coordinated cutover runbook](https://github.com/NotebookNomad/icepick-agent/blob/main/docs/repository-rename.md)
lives in the paired agent PR and becomes available on `main` after that PR merges.

## Release scope

The README, clone URL, help heading and consumer links use the new repository
names. `deck`, its arguments, Dockerfile, image tag (`icepick-offsec:latest`),
Compose configuration, mount paths, firewall settings and test dependency pins
stay unchanged. Existing sessions are still disposable. The backend does not yet
provide the shared MCP runtime or autonomous lifecycle: those will be extracted
from icepick-agent in later PRs. The agent keeps its current backend pin for this
rename, so it does not automatically adopt this repository's newer changes.

## Existing manual checkouts

Keep the directory in place. After the GitHub rename, update only its remote, then
fetch and update your branch normally once the preparation PR is merged:

```bash
git remote set-url origin https://github.com/NotebookNomad/icepick-backend.git
git fetch origin
```

For SSH, use `git@github.com:NotebookNomad/icepick-backend.git`. Preserve existing
transport preferences and local URL rewrites. Record `docker compose config`
before updating and compare its project, volumes and mounts afterwards with the
same environment. Do not stop a session, rebuild an image or recreate volumes to
apply a GitHub rename. Avoid publishing the full config if it contains local
settings you do not want to share.

Compose currently derives this repository's default project name from its
checkout directory. That name also prefixes persistent volumes such as
`icepick-offsec_toolconfig`. Renaming the directory to `icepick-backend` would
select different volumes; existing credentials and caches could appear missing.
Keep any existing `COMPOSE_PROJECT_NAME`, `.env` or `-p` override unchanged as well.
The README therefore uses the new URL with the explicit legacy clone destination
`icepick-offsec`. A filesystem/project migration requires a separate plan.

Do not change `/root/workspace`, move `workspace/` or `vpn/`, or run
`deck clean --volumes` during this migration. There is no data migration and no
new image tag to pull or build.

## Maintainer verification

Rename this GitHub repository first, then the agent repository. Verify new remote
URLs and existing PR/release history before merging the paired preparation PRs.
Update the backend description to:

> Kali execution backend for authorized security labs: manual deck workflows; shared MCP runtime extraction in progress.

Run `./tests/run.sh static unit` with the test submodules initialized and confirm
the existing GitHub Actions test workflow works under the new repository name.
Compare Compose output with the same project path and environment; Dockerfile,
Compose and test submodule pins must be unchanged. Verify fresh clones and links
once both new URLs are live. No live VPN or image rebuild is needed for these
reference changes, and they do not establish new networking guarantees.

If either new URL fails, keep both preparation PRs unmerged and repair or revert
the repository settings change. After merge, a reference rollback reverts the
preparation commits and restores the recorded remote only if it resolves; it
never deletes evidence, credentials, volumes or images. Do not reuse the old
GitHub repository names for unrelated repositories, so redirects remain usable.
