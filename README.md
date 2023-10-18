# Go bindings to SQLite using Wazero

[![Go Reference](https://pkg.go.dev/badge/image)](https://pkg.go.dev/github.com/ncruces/go-sqlite3)
[![Go Report](https://goreportcard.com/badge/github.com/ncruces/go-sqlite3)](https://goreportcard.com/report/github.com/ncruces/go-sqlite3)
[![Go Coverage](https://github.com/ncruces/go-sqlite3/wiki/coverage.svg)](https://github.com/ncruces/go-sqlite3/wiki/Test-coverage-report)

Go module `github.com/ncruces/go-sqlite3` wraps a [WASM](https://webassembly.org/) build of [SQLite](https://sqlite.org/),
and uses [wazero](https://wazero.io/) to provide `cgo`-free SQLite bindings.

- [`github.com/ncruces/go-sqlite3`](https://pkg.go.dev/github.com/ncruces/go-sqlite3)
  wraps the [C SQLite API](https://www.sqlite.org/cintro.html)
  ([example usage](https://pkg.go.dev/github.com/ncruces/go-sqlite3#example-package)).
- [`github.com/ncruces/go-sqlite3/driver`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/driver)
  provides a [`database/sql`](https://pkg.go.dev/database/sql) driver
  ([example usage](https://pkg.go.dev/github.com/ncruces/go-sqlite3/driver#example-package)).
- [`github.com/ncruces/go-sqlite3/embed`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/embed)
  embeds a build of SQLite into your application.
- [`github.com/ncruces/go-sqlite3/vfs`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/vfs)
  wraps the [C SQLite VFS API](https://www.sqlite.org/vfs.html) and provides a pure Go implementation.
- [`github.com/ncruces/go-sqlite3/vfs/memdb`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/vfs/memdb)
  implements an in-memory VFS.
- [`github.com/ncruces/go-sqlite3/vfs/readervfs`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/vfs/readervfs)
  implements a VFS for immutable databases.
- [`github.com/ncruces/go-sqlite3/ext/unicode`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/ext/unicode)
  registers Unicode aware functions.
- [`github.com/ncruces/go-sqlite3/ext/stats`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/ext/stats)
  registers [statistics functions](https://www.oreilly.com/library/view/sql-in-a/9780596155322/ch04s02.html).
- [`github.com/ncruces/go-sqlite3/gormlite`](https://pkg.go.dev/github.com/ncruces/go-sqlite3/gormlite)
  provides a [GORM](https://gorm.io) driver.

### Caveats

This module replaces the SQLite [OS Interface](https://www.sqlite.org/vfs.html)
(aka VFS) with a [pure Go](vfs/) implementation.
This has benefits, but also comes with some drawbacks.

#### Write-Ahead Logging

Because WASM does not support shared memory,
[WAL](https://www.sqlite.org/wal.html) support is [limited](https://www.sqlite.org/wal.html#noshm).

To work around this limitation, SQLite is [patched](sqlite3/locking_mode.patch)
to always use `EXCLUSIVE` locking mode for WAL databases.

Because connection pooling is incompatible with `EXCLUSIVE` locking mode,
to use the [`database/sql`](https://pkg.go.dev/database/sql)
driver with WAL mode databases you should disable connection pooling by calling
[`db.SetMaxOpenConns(1)`](https://pkg.go.dev/database/sql#DB.SetMaxOpenConns).

#### File Locking

POSIX advisory locks, which SQLite uses on Unix, are
[broken by design](https://www.sqlite.org/src/artifact/2e8b12?ln=1073-1161).

On Linux, macOS and illumos, this module uses
[OFD locks](https://www.gnu.org/software/libc/manual/html_node/Open-File-Description-Locks.html)
to synchronize access to database files.
OFD locks are fully compatible with POSIX advisory locks.

On BSD Unixes, this module uses
[BSD locks](https://man.freebsd.org/cgi/man.cgi?query=flock&sektion=2).
On BSD Unixes, BSD locks are fully compatible with POSIX advisory locks.

On Windows, this module uses `LockFile`, `LockFileEx`, and `UnlockFile`, like SQLite.

On all other platforms, file locking is not supported, and you must use
[`nolock=1`](https://www.sqlite.org/uri.html#urinolock)
to open database files.

#### Testing

The pure Go VFS is tested by running SQLite's
[mptest](https://github.com/sqlite/sqlite/blob/master/mptest/mptest.c)
on Linux, macOS, Windows and FreeBSD.
Performance is tested by running
[speedtest1](https://github.com/sqlite/sqlite/blob/master/test/speedtest1.c).

### Roadmap

- [ ] advanced SQLite features
  - [x] custom functions
  - [x] nested transactions
  - [x] incremental BLOB I/O
  - [x] online backup
  - [x] JSON support
  - [ ] session extension
- [ ] custom VFSes
  - [x] custom VFS API
  - [x] in-memory VFS
  - [x] read-only VFS, wrapping an [`io.ReaderAt`](https://pkg.go.dev/io#ReaderAt)
  - [ ] cloud-based VFS, based on [Cloud Backed SQLite](https://sqlite.org/cloudsqlite/doc/trunk/www/index.wiki)

### Alternatives

- [`modernc.org/sqlite`](https://pkg.go.dev/modernc.org/sqlite)
- [`crawshaw.io/sqlite`](https://pkg.go.dev/crawshaw.io/sqlite)
- [`github.com/mattn/go-sqlite3`](https://pkg.go.dev/github.com/mattn/go-sqlite3)
- [`github.com/zombiezen/go-sqlite`](https://pkg.go.dev/github.com/zombiezen/go-sqlite)
