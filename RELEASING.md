# Releasing Modelplane

This is the maintainer process for cutting a Modelplane release and versioning
the docs. Contributing changes is covered in [CONTRIBUTING.md](CONTRIBUTING.md);
this file is only for the small set of people who publish releases.

## Releasing

Releases are cut from a release branch and published by the `CI` workflow. To
release a new minor version, e.g. `v0.1.0`:

1. From the GitHub UI, create a `release-0.1` branch from `main`.
2. Create a GitHub release targeting that branch, and let the release create the
   tag `v0.1.0`.
3. Run the `CI` workflow (Actions → CI → Run workflow) against the `v0.1.0` tag,
   setting the `tag` input to `v0.1.0`.

The `tag` input makes the workflow push the package with that exact version
rather than the dev version it derives from git metadata on ordinary runs.
Patch releases (e.g. `v0.1.1`) reuse the existing `release-0.1` branch: cut the
release from it and run the workflow against the new tag.

## Versioning the docs

Docs are versioned at the minor level, and one Vercel project in the
[docs-site](https://github.com/modelplaneai/docs-site) repo serves every
version. Its build clones one branch of this repo per version: the release
named `latest` is served at the domain root, the others under a path prefix.

| URL | Built from |
|---|---|
| `docs.modelplane.ai/…` | this repo's current `release-X.Y` branch |
| `docs.modelplane.ai/main/…` | this repo's `main` |
| `docs.modelplane.ai/vX.Y/…` | this repo's older `release-X.Y` branches |

The version list is `themes/geekboot/data/docversions.json` on the site repo's
`main` branch, and it is the only place any of this is written down. Each entry
names a version, its path prefix, and the branch of this repo it builds from;
nothing is committed per release branch here, so cutting `release-0.4` needs no
docs change in this repo.

To publish docs for a new minor release (e.g. `v0.4.0`), once `release-0.4`
exists here (step 1 of Releasing above):

1. In the site repo, add the release to the version list, newest first:
   ```json
   { "version": "0.4", "path": "v0.4", "branch": "release-0.4" }
   ```
2. Set `"latest": "0.4"` in the same file, so the new release moves to the
   domain root and the previous one moves to its own `/v0.3/` prefix, picking
   up the "older version" banner.
3. Merge. The site rebuilds every version, the switcher offers the new one, and
   the DocSearch crawl repoints at the new root.

No Vercel project, domain, or DNS record is created per release, and a missing
branch fails the whole site build, so add the entry only after the branch
exists.

To fix a typo in an archived version, push to that `release-X.Y` branch here.
The site reads each branch's tip at build time but nothing here triggers that
build, so redeploy the site project from the Vercel dashboard to publish it.
