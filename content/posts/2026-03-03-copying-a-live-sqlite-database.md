---
title: copying a live sqlite database
date: 2026-03-03
tags: [tools]
---

SQLite is the perfect tool for those cases where you need structured data but
don't want to go with a full-blown database like Postgres. The entire database
is just a file on disk making it super easy to work with.

A common thing you'll want to be able to do is to make a copy of a SQLite
database, such as for backup, or to pull it down from a server to do something
with the data. Being a file, you might want to just copy it directly:

```sh
cp live.db copy.db
```

But if the database is live and in use, this will likely result in a corrupted
copy. The correct way to copy a live database is to use the CLI tool instead:

```sh
sqlite3 live.db '.backup copy.db'
```

If the live database is under heavy use however, you may get back

```
Error: database is locked
```

in which case you can set a timeout for SQLite to wait for the database to
become available:

```sh
sqlite3 live.db '.timeout 3000' '.backup copy.db'
```
