# Changelog

Every release of apogee-storage, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.1 (2026-11-27)

### Fixes

- Database connections that the server dropped while idle (MySQL's `wait_timeout`, a restart, a proxy's idle limit) reconnect on their next use instead of failing with "server has gone away" (grav-db 0.2.1). Uses the new releases of apogee-crypt, apogee-package.

## 0.2.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- `StoragePackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring moved here from the framework.

### Dependencies

- Uses apogee-package 0.1.0, apogee-crypt 0.2.0 (from their GitHub tags).

## 0.1.2 (2026-11-21)

- The S3 date helper is `S3Dates` (it was `Civil`, which apogee-support also
  defines, so the two could not be in one program).

## 0.1.1 (2026-11-21)

### Performance

- An S3 disk keeps a small pool of keep-alive HTTP clients instead of one
  behind a lock, so requests from several threads run side by side: 320
  uploads from 8 threads against a server with 20 ms of latency took 0.8 s
  instead of 6.5 s.

## 0.1.0 (2026-11-20)

The first release. Needs grav 1.10.0 or newer.

- `Disk`, with the memory, local and S3 disks, and the `Storage` facade with a
  fake for tests.
- S3: Signature Version 4, presigned temporary URLs, paged listings, copies,
  temporary credentials, public URLs through a CDN; Amazon and S3-compatible
  services.
- Local: atomic writes, public URLs, and temporary URLs signed with
  apogee-crypt.
