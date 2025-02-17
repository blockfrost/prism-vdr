# PRISM VDR Indexer

TODO

## mainnet

`mainnet` have :
- 6455 Transactions
- 9831 PRISM events/operations
- 312 SSI entries

## preprod

Running from scratch it took `Total time: 375 s (06:15), completed Feb 11, 2025, 8:55:59 PM`
Most of the time was FileSystem IO and also a bit of Network IO.

`preprod` have :
- 16938 Transactions
- 275569 PRISM events/operations
- 71408 SSI entries

```shell
➜  preprod git:(main) ✗ du -h     
1.0G	./ssi
1.0G	./diddoc
1.0G	./ops
1.0G	./opid
4.5G	.

➜  preprod git:(main) ✗ ls -lah
total 740648
drwxr-xr-x      9 fabio  staff   288B Feb 11 20:49 .
drwxr-xr-x      7 fabio  staff   224B Feb 11 18:32 ..
-rw-r--r--      1 fabio  staff   117M Feb 11 18:58 cardano-21325-cbor
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 diddoc
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 opid
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 ops
-rw-r--r--      1 fabio  staff   244M Feb 11 20:50 prism-events
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 ssi
```

Using the normal FileSystem as storage is also super inefficient.
From 8.3 MB of uncompressed text to 1.0G in used space... This could also be optimized with special file types.
Hopefully git is very efficient. So this use case is fine. Although there are Git commands that we should avoid because they take time.
But the GitHub interface would probably complain about the number of files. We could have a hashing table structure like using folders for this!

The idea of using GitHub Actions may be more complicated because of this.
Probably we need to do some optimizations in terms of FileSystem or craft the cache of the action very carefully.


We also have the problem of large files
I don't want to use Git LFS (is not a good use case). Since those files are not assets they are mutable datasets!
So the solution will be to have logic to split file in code.

```shell
-rw-r--r--      1 fabio  staff    42M Feb 11 22:09 cardano-21325-cbor-a
-rw-r--r--      1 fabio  staff    75M Feb 11 22:10 cardano-21325-cbor-b
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 diddoc
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 opid
drwxr-xr-x  65535 fabio  staff   8.3M Feb 11 20:55 ops
-rw-r--r--      1 fabio  staff    44M Feb 11 22:19 prism-events-a
-rw-r--r--      1 fabio  staff    44M Feb 11 22:19 prism-events-b
-rw-r--r--      1 fabio  staff    44M Feb 11 22:19 prism-events-c
-rw-r--r--      1 fabio  staff    44M Feb 11 22:19 prism-events-d
-rw-r--r--      1 fabio  staff    44M Feb 11 22:19 prism-events-e
-rw-r--r--      1 fabio  staff    23M Feb 11 22:19 prism-events-f
```

## TODO

- Maybe merge folder `opid` and `ops` => opid just have the extra hash of the operation.
  But the index is the same.
- Fetch relevant transactions and store info about it.
- split the raw files (support in code):
  - cardano-21325-cbor
  - prism-events
- Remove the support for cbor metadata and that encodes PRISM operation using text (hex).
- Support a hot index start. (Right now, it only starts from scratch)
- PoC for a GitHub Action
- The folder diddoc have the DID Document but maybe would be more useful to replace with the did-resolution-result https://w3c.github.io/did-resolution/#did-resolution-result
  - the field `didResolutionMetadata` ... will be contraceptive that can be empty
  - the field `didDocumentMetadata` can have many useful fields: https://w3c.github.io/did-resolution/#did-document-metadata
    - created, updated, deactivated
- support for transactions with multiple labels (assuming this is possible)
- Missing Index is the OperationHash -> Previous Operation Hash (if existing)

### Github Action

- Cache - https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/caching-dependencies-to-speed-up-workflows#usage-limits-and-eviction-policy
- Cache - https://graphite.dev/guides/github-actions-caching
- Docker - https://aschmelyun.com/blog/using-docker-run-inside-of-github-actions/

## Folder structure utils

```shell
#rm -rf cardano-21325
rm -rf prism-events
rm -rf diddoc
rm -rf ops
rm -rf ssi

mkdir cardano-21325
mkdir prism-events
mkdir diddoc
mkdir ops
mkdir ssi
```