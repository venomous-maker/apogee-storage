# Changelog

Every release of apogee-storage, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `Disk`, with the memory, local and S3 disks, and the `Storage` facade with a
  fake for tests.
- S3: Signature Version 4, presigned temporary URLs, paged listings, copies,
  temporary credentials, public URLs through a CDN; Amazon and S3-compatible
  services.
- Local: atomic writes, public URLs, and temporary URLs signed with
  apogee-crypt.
