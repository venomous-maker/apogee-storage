# apogee-storage

Files for [Apogee](https://github.com/venomous-maker) apps: one API over a
folder on this machine, an S3-compatible bucket (Amazon S3, Cloudflare R2,
Backblaze B2, DigitalOcean Spaces, SeaweedFS, MinIO, ...), or memory for
tests. Public URLs, temporary URLs for private files, and a fake for tests.

```grav
import "std"
import "apogee-storage"
use apogee.*

Storage.addDisk("local", new LocalDisk("storage/app", "https://example.com/files", env("APP_KEY")))
Storage.addDisk("s3", new S3Disk(S3Config.aws("my-bucket", "eu-west-1", env("AWS_KEY"), env("AWS_SECRET"))))

Storage.disk("s3").putFile("avatars/42.png", upload.tempPath)
let link = Storage.disk("s3").temporaryUrl("avatars/42.png", 300)   // works for five minutes
let text = Storage.get("notes/today.txt")                            // the default disk (the first added)
```

## In an apogee-framework app

Importing the module is all it takes: `StoragePackage` (apogee-package) connects it when
the app is created. The `local` and `public` disks are added under the app folder, and `s3` when a bucket is set; `filesystems.default` is the default disk. Disks sign URLs with the app key when it is long enough.

| Setting | Default |
|---|---|
| `filesystems.default` | `FILESYSTEM_DISK`, `local` |
| `filesystems.disks.local.root` | `storage/app` |
| `filesystems.disks.public.root`, `.url` | `storage/app/public`, `APP_URL` + `/storage` |
| `filesystems.disks.s3.bucket`, `.region`, `.key`, `.secret` | `AWS_BUCKET`, `AWS_DEFAULT_REGION` (`us-east-1`), `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| `filesystems.disks.s3.endpoint`, `.path_style` | `AWS_ENDPOINT` (another S3-compatible service), `AWS_USE_PATH_STYLE_ENDPOINT` (on) |

Your own configuration wins over these defaults.

## Disks

| | |
|---|---|
| `new LocalDisk(root, baseUrl = "", signingKey = "")` | a folder; writes go to a temporary file renamed into place, so readers never see half a file |
| `new S3Disk(S3Config.aws(bucket, region, key, secret))` | an Amazon S3 bucket |
| `new S3Disk(S3Config.compatible(endpoint, bucket, key, secret, region = "us-east-1"))` | another S3-compatible service (path-style) |
| `new MemoryDisk()` | in memory |

`S3Config` also takes `sessionToken` (temporary credentials), `publicUrl` (a
CDN in front of the bucket) and `pathStyle`. `S3Disk.createBucket()` makes the
bucket.

## Every disk

| | |
|---|---|
| `put(path, text)`, `putBytes(path, bytes)`, `putFile(path, localFile)` | store (directories as needed) |
| `get(path)`, `getBytes(path)`, `getOr(path, fallback)`, `download(path, localFile)` | read; `FileNotFoundError` when missing |
| `exists`, `missing`, `size`, `lastModified`, `mimeType` | |
| `delete(path)`, `deleteMany(paths)`, `deleteDirectory(dir)` | |
| `copy(from, to)`, `move(from, to)`, `append(path, text)`, `prepend(path, text)` | |
| `files(dir, recursive)`, `allFiles(dir)`, `directories(dir)` | paths from the top of the disk, sorted |
| `url(path)` | the public URL |
| `temporaryUrl(path, ttlSeconds)` | S3: a presigned GET. Local: `url` plus `expires` and a signature (made with apogee-crypt); the route serving the files checks it with `verifyTemporaryUrl(path, expires, signature)` |

Paths are relative, use `/`, and cannot climb out of the disk (`..` is a
`ValueError`).

## The facade

`Storage.addDisk(name, disk)` (the first is the default), `Storage.disk(name)`,
`Storage.setDefault(name)`, and the default disk's `put`/`get`/`exists`/
`delete`/`url` as statics. In tests, `Storage.fake("s3")` puts an empty memory
disk in place of `"s3"` and returns it.

## How S3 requests are signed

AWS Signature Version 4, checked against AWS's published example; the S3 tests
run against a server that verifies every signature. Listings follow
continuation tokens past 1,000 keys.

## Tests

```bash
grav test tests/storage.grav        # needs nothing
grav test tests/package.grav
grav test tests/s3.grav             # needs an S3-compatible server
```

`tests/s3.grav` reads `S3_ENDPOINT`, `S3_KEY` and `S3_SECRET`; by default it
expects SeaweedFS on port 8333 with these credentials:

```bash
echo '{"identities":[{"name":"test","credentials":[{"accessKey":"AKIDGRAVTEST","secretKey":"gravSecretKeyForTests123"}],"actions":["Admin","Read","Write","List","Tagging"]}]}' > /tmp/s3.json
docker run -d -p 8333:8333 -v /tmp/s3.json:/etc/s3.json chrislusf/seaweedfs server -s3 -s3.config=/etc/s3.json
```
