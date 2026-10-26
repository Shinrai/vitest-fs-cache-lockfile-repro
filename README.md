# Vitest fs module cache: packages with their own lockfile keep wiping each other's cache

`packages/app` is an npm workspace member and uses the root `package-lock.json`. `packages/tool` sits inside the same repo but isn't a workspace member, so it has its own `package-lock.json`. Vitest puts both packages' fs module cache in the same place (`node_modules/.vitest-cache` at the repo root), but each one hashes a different lockfile.

## Steps

```sh
npm install
(cd packages/tool && npm install)

(cd packages/app && npx vitest run)    # fills the cache
(cd packages/app && npx vitest run)    # reuses it
(cd packages/tool && npx vitest run)   # "fs cache was cleared because lockfile has changed"
(cd packages/app && npx vitest run)    # cleared again
```

Run with `DEBUG=vitest:cache:fs` to see the message. Nothing was installed between runs, but every switch between the two packages clears the whole shared cache.
