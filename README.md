# Repro for turborepo with no repository caching issue


The project is a monorepo with 2 packages where package `ws-2` depends on package `ws-1`.

```
▲ tree --gitignore
.
├── README.md
├── package.json
├── turbo.json
├── workspaces
│   ├── ws-1
│   │   ├── package.json
│   │   ├── src
│   │   │   └── index.ts
│   │   └── tsconfig.json
│   └── ws-2
│       ├── package.json
│       ├── src
│       │   └── index.ts
│       └── tsconfig.json
└── yarn.lock
```

To reproduce the issue clone this repository

- `git clone https://github.com/antonk52/turbo-caching-issue`
- `cd turbo-caching-issue`
- remove `.git` as the issue only reproduces with no VCS<br/>`rm -rf .git`
- `yarn` to install dependencies
- build `ws-2` and its dependencies first time to populate cache
  - `yarn run build --filter=ws-2...`
- remove all build artifacts
  - `yarn run clean`
- build `ws-2` and its dependencies again to see `FULL TURBO`
  - `yarn run build --filter=ws-2...`
- make a change to `ws-1` source code to **invalidate** cache
  - `echo "export function anotherOne() {};" >> workspaces/ws-1/src/index.ts`
- build `ws-2` and its dependencies third time with expectation of cache miss and instead see `FULL TURBO`
  - `yarn run build --filter=ws-2...`

Full output is presented below:

```
~/Documents/dev/personal/turbo-caching-issue
▲ yarn run clean
yarn run v1.22.19
$ turbo run clean
 WARNING  cannot find a .git folder. Falling back to manual file hashing (which may be slower). If you are running this build in a pruned directory, you can ignore this message. Otherwise, please initialize a git repository in the root of your monorepo
• Packages in scope: ws-1, ws-2
• Running clean in 2 packages
ws-1:clean: cache bypass, force executing 23c288c6810b3863
ws-2:clean: cache bypass, force executing 23c288c6810b3863
ws-1:clean: $ rm -rf dist
ws-2:clean: $ rm -rf dist

 Tasks:    2 successful, 2 total
Cached:    0 cached, 2 total
  Time:    508ms

✨  Done in 0.68s.

~/Documents/dev/personal/turbo-caching-issue
▲ yarn run build --filter=ws-2...
yarn run v1.22.19
$ turbo run build --filter=ws-2...
 WARNING  cannot find a .git folder. Falling back to manual file hashing (which may be slower). If you are running this build in a pruned directory, you can ignore this message. Otherwise, please initialize a git repository in the root of your monorepo
• Packages in scope: ws-1, ws-2
• Running build in 2 packages
ws-1:build: cache miss, executing 9379ffc4843bb409
ws-1:build: $ tsc
ws-2:build: cache miss, executing 2c25b27926080a2d
ws-2:build: $ tsc

 Tasks:    2 successful, 2 total
Cached:    0 cached, 2 total
  Time:    1.826s

✨  Done in 1.97s.

~/Documents/dev/personal/turbo-caching-issue
▲ yarn run build --filter=ws-2...
yarn run v1.22.19
$ turbo run build --filter=ws-2...
 WARNING  cannot find a .git folder. Falling back to manual file hashing (which may be slower). If you are running this build in a pruned directory, you can ignore this message. Otherwise, please initialize a git repository in the root of your monorepo
• Packages in scope: ws-1, ws-2
• Running build in 2 packages
ws-1:build: cache hit, replaying output 9379ffc4843bb409
ws-1:build: $ tsc
ws-2:build: cache hit, replaying output 2c25b27926080a2d
ws-2:build: $ tsc

 Tasks:    2 successful, 2 total
Cached:    2 cached, 2 total
  Time:    204ms >>> FULL TURBO

✨  Done in 0.33s.

~/Documents/dev/personal/turbo-caching-issue
▲ echo "export function anotherOne() {};" >> workspaces/ws-1/src/index.ts

~/Documents/dev/personal/turbo-caching-issue
▲ yarn run build --filter=ws-2...
yarn run v1.22.19
$ turbo run build --filter=ws-2...
 WARNING  cannot find a .git folder. Falling back to manual file hashing (which may be slower). If you are running this build in a pruned directory, you can ignore this message. Otherwise, please initialize a git repository in the root of your monorepo
• Packages in scope: ws-1, ws-2
• Running build in 2 packages
ws-1:build: cache hit, replaying output 9379ffc4843bb409
ws-1:build: $ tsc
ws-2:build: cache hit, replaying output 2c25b27926080a2d
ws-2:build: $ tsc

 Tasks:    2 successful, 2 total
Cached:    2 cached, 2 total
  Time:    216ms >>> FULL TURBO

✨  Done in 0.36s.

```
