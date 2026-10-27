# thinstation-files

Private binary storage for the [thinstation-ng](https://github.com/sajjadsabzkar/thinstation-ng)
build. Nothing here is a source file: these are vendor installers and the frozen
staged Horizon tree, and they are the reason a CI build can reproduce exactly the
image that was boot-proven by hand.

The build downloads them over a URL with a token, so they live on **Release
assets**, not in git - a 337 MB blob in a commit would be unrecoverable by any
normal means and is over GitHub's file limit anyway.

## Contents

| release tag | file | size | sha256 |
| --- | --- | --- | --- |
| `vendor-20261027` | `thinstation-vendor-20261027.tar.zst` | 337,876,785 | `8a5ddfb84accdb0e12633772d70f717199ce027be87de204dd464d1235c9d83e` |

The tarball is `ts/build/packages/horizon/{bin,etc,lib64}` plus a `VENDOR.txt`
provenance note. Make a new one with `docker/ts-vendor.sh make` in the build
repository, upload it as a release asset here, then update the
`HORIZON_VENDOR_SHA256` variable in the build repository to the sha256 `make`
prints. CI refuses to restore an archive whose checksum differs from that pin.

## How the build reaches it

`HORIZON_VENDOR_URL` points at a release asset's download URL and
`HORIZON_VENDOR_TOKEN` is a token with read access to this repository. The asset
is not public; an unauthenticated request gets 404, not a redirect.
