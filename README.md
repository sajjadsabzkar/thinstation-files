# thinstation-files

Binary storage for the [thinstation-ng](https://github.com/sajjadsabzkar/thinstation-ng)
build. Nothing here is a source file: these are vendor installers and the frozen
staged Horizon tree, and they are the reason a CI build can reproduce exactly the
image that was boot-proven by hand.

They are fetched over an ordinary URL, so they live on **Release assets** and not
in git - a 337 MB blob in a commit would be unrecoverable by any normal means and
is over GitHub's file limit anyway.

## Contents

| file | size | sha256 |
| --- | --- | --- |
| `thinstation-vendor-20261027.tar.zst` | 337,876,785 | `8a5ddfb84accdb0e12633772d70f717199ce027be87de204dd464d1235c9d83e` |

Release tag: `vendor-20261027`. The URL a build is given is the release page's
plain download link - no asset id, no special header, no credential:

```
https://github.com/sajjadsabzkar/thinstation-files/releases/download/vendor-20261027/thinstation-vendor-20261027.tar.zst
```

That string is what `vendor_url` says in the flavor file of the build repository
(`ts/build/flavors/horizon5.env`), and the sha256 above is its `vendor_sha256`
pin. Nothing about this download is configured in that repository's Settings: the
URL and the pin live in git, where a diff shows both.

## Why this repository is public, and what still protects the payload

The archive is vendor binaries, kept here rather than in the build repository so
the build stays a normal checkout. It is readable by anyone who has the URL. That
is a deliberate trade, because the alternative - a private release asset - cannot
be made to work for more than one image, and one image per token is exactly what
the flavor design refuses to be.

Measured while this repository was private: `releases/download/TAG/FILE` answers
**404 to every token form**, even a valid one, and the only thing that streams
bytes is the asset's API endpoint
(`https://api.github.com/repos/OWNER/REPO/releases/assets/ASSET_ID`) requested with
`Accept: application/octet-stream` - without that header the response is the
asset's JSON metadata at status 200, which downloads cleanly and then fails its
checksum, looking exactly like a corrupt archive. That endpoint is named by an
asset id, and the id, the header and the bearer token all have to come from
repository secrets, which every workflow in the repository shares: one archive,
ever. A second flavor with a second Horizon version could not have had a URL of
its own.

So the URL is open and the pin is what carries the trust. Whoever can replace the
file at that address still cannot change what a build restores - the download step
compares the sha256 of what arrived against the pin written in the flavor file and
fails the build on a mismatch, and `docker/ts-vendor.sh restore` checks the same
comparison again before it writes anything into the tree. Substituting a payload
would mean editing a tracked file, which shows up in a diff and in the release's
own provenance note.

## Making a new archive

Run `docker/ts-vendor.sh make` in the build repository. It prints a sha256; upload
the file it wrote here as an asset on a release, then write both values - the
download link and that sha256 - into `vendor_url` and `vendor_sha256` of every
`ts/build/flavors/<name>.env` that ships this payload. `make` prints the
`HORIZON_VENDOR_SHA256=<pin> docker/ts-vendor.sh restore <path>` line to check the
archive restores cleanly before it is hosted, and `docker/ts-vendor.sh show
<path>` prints its provenance, including the md5 of the two vendor installers it
was made from.

The tarball is `ts/build/packages/horizon/{bin,etc,lib64}` plus a `VENDOR.txt`
provenance note, which is what `restore` unpacks back into the build tree.
