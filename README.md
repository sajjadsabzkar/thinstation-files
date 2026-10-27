# thinstation-files

Private binary storage for the [thinstation-ng](https://github.com/sajjadsabzkar/thinstation-ng)
build. Nothing here is a source file: these are vendor installers and the frozen
staged Horizon tree, and they are the reason a CI build can reproduce exactly the
image that was boot-proven by hand.

The build downloads them over a URL with a token, so they live on **Release
assets**, not in git - a 337 MB blob in a commit would be unrecoverable by any
normal means and is over GitHub's file limit anyway.

## Contents

| asset id | file | size | sha256 |
| --- | --- | --- | --- |
| `680747193` | `thinstation-vendor-20261027.tar.zst` | 337,876,785 | `8a5ddfb84accdb0e12633772d70f717199ce027be87de204dd464d1235c9d83e` |

Release tag: `vendor-20261027`. The URL the build is given is the asset's API
form, not the release page's download link:

```
https://api.github.com/repos/sajjadsabzkar/thinstation-files/releases/assets/680747193
```

That is not interchangeable. Measured against a private asset:
`releases/download/TAG/FILE` answers **404 to every token form**, even a valid one,
while the API endpoint redirects to a signed blob and streams the bytes. The API
form also needs `Accept: application/octet-stream`, because its default response is
the asset's JSON metadata with status 200 - which downloads cleanly and then fails a
checksum, looking exactly like a corrupt archive.

The tarball is `ts/build/packages/horizon/{bin,etc,lib64}` plus a `VENDOR.txt`
provenance note. Make a new one with `docker/ts-vendor.sh make` in the build
repository, upload it as a release asset here, then update the
`HORIZON_VENDOR_SHA256` variable in the build repository to the sha256 `make`
prints. CI refuses to restore an archive whose checksum differs from that pin.

## How the build reaches it

`HORIZON_VENDOR_URL` holds the URL above and `HORIZON_VENDOR_TOKEN` is a token that
can read this repository. The asset is not public: unauthenticated, or authenticated
against the wrong URL form, the answer is 404 rather than a redirect. Note that
`GITHUB_TOKEN` inside a build is scoped to the build repository only, so a
cross-repository download always needs its own secret.
