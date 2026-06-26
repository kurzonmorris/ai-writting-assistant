# `server/storage/` — Cloud Storage Integration

Connects to the writer's **own** cloud storage, which is the **permanent memory**
of the system (see `../../../../docs/06-cloud-storage-integration.md`). Google
Drive is the first provider.

## The interface (why it's pluggable)

All cloud-storage code sits behind one small internal interface so the rest of the
app doesn't care which provider is used:

```
connect()        → run the OAuth flow, store an encrypted token + a StorageConnection
disconnect()     → revoke and delete the stored token
createFolder()   → make the app's root/project folders in the writer's storage
listFiles()      → list documents the app manages
readFile()       → read a document's contents
writeFile()      → write/refresh a source doc or a knowledge-base file
```

Google Drive is one implementation. Adding Dropbox or OneDrive later means writing
another implementation of the same interface, with no changes elsewhere.

## Security notes (critical)

- Request the **minimum OAuth scope** that works — ideally app-folder-only so we
  can't see the writer's whole Drive (doc 06, OPEN question Q2).
- Tokens are **encrypted at rest** (doc 05) using `TOKEN_ENCRYPTION_KEY`; they are
  decrypted only here, in memory, when needed.
- Tokens never go to the browser or to AI providers.

## What we write to the writer's storage

- `source/` — the original imported documents.
- `knowledge-base/` — the compiled, human-readable story bible (doc 06).

## Status

Plan only. Built in Phase 4 (doc 11); Phase 1 uses internal storage only.
