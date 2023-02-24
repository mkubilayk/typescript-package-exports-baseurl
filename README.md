# Reproduction for microsoft/TypeScript#52950

https://github.com/microsoft/TypeScript/issues/52950

First build the producer:

```sh
npm install
npx tsc -p packages/producer-with-package-exports
```

Test the consumer without `baseUrl`:

```sh
npx tsc -p packages/consumer-without-baseurl
# all imports are resolved correctly, works as expected ✅
```

Test the consumer with `baseUrl`:

```sh
npx tsc -p packages/consumer-with-baseurl
# fails to resolve the import using package exports 💥
```

Get the resolution trace:

```sh
npx tsc -p packages/consumer-with-baseurl --traceResolution
```

✅ Good (`producer-with-package-exports`):

```
======== Resolving module 'producer-with-package-exports' from '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/index.ts'. ========
Explicitly specified module resolution kind: 'Node16'.
Resolving in CJS mode with conditions 'node', 'require', 'types'.
'baseUrl' option is set to '/redacted/typescript-package-exports-baseurl/packages', using this value to resolve non-relative module name 'producer-with-package-exports'.
Resolving module name 'producer-with-package-exports' relative to base url '/redacted/typescript-package-exports-baseurl/packages' - '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports'.
Loading module as file / folder, candidate module location '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports', target file type 'TypeScript'.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports.ts' does not exist.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports.tsx' does not exist.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports.d.ts' does not exist.
Found 'package.json' at '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/package.json'.
'package.json' does not have a 'typesVersions' field.
'package.json' does not have a 'typings' field.
'package.json' has 'types' field 'out/index.d.ts' that references '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/out/index.d.ts'.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/out/index.d.ts' exist - use it as a name resolution result.
======== Module name 'producer-with-package-exports' was successfully resolved to '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/out/index.d.ts' with Package ID 'producer-with-package-exports/out/index.d.ts@1.0.0'. ========
```

💥 Failure (`producer-with-package-exports/entrypoint-a`):

```
======== Resolving module 'producer-with-package-exports/entrypoint-a' from '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/index.ts'. ========
Explicitly specified module resolution kind: 'Node16'.
Resolving in CJS mode with conditions 'node', 'require', 'types'.
'baseUrl' option is set to '/redacted/typescript-package-exports-baseurl/packages', using this value to resolve non-relative module name 'producer-with-package-exports/entrypoint-a'.
Resolving module name 'producer-with-package-exports/entrypoint-a' relative to base url '/redacted/typescript-package-exports-baseurl/packages' - '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a'.
Loading module as file / folder, candidate module location '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a', target file type 'TypeScript'.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a.ts' does not exist.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a.tsx' does not exist.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a.d.ts' does not exist.
Directory '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a' does not exist, skipping all lookups in it.
File '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/package.json' does not exist according to earlier cached lookups.
File '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/package.json' exists according to earlier cached lookups.
Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file type 'TypeScript'.
Directory '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/typescript-package-exports-baseurl/packages/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/typescript-package-exports-baseurl/node_modules/@types' does not exist, skipping all lookups in it.
Directory '/redacted/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
'baseUrl' option is set to '/redacted/typescript-package-exports-baseurl/packages', using this value to resolve non-relative module name 'producer-with-package-exports/entrypoint-a'.
Resolving module name 'producer-with-package-exports/entrypoint-a' relative to base url '/redacted/typescript-package-exports-baseurl/packages' - '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a'.
Loading module as file / folder, candidate module location '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a', target file type 'JavaScript'.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a.js' does not exist.
File '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a.jsx' does not exist.
Directory '/redacted/typescript-package-exports-baseurl/packages/producer-with-package-exports/entrypoint-a' does not exist, skipping all lookups in it.
File '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/package.json' does not exist according to earlier cached lookups.
File '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/package.json' exists according to earlier cached lookups.
Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file type 'JavaScript'.
Directory '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/src/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/typescript-package-exports-baseurl/packages/consumer-with-baseurl/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/typescript-package-exports-baseurl/packages/node_modules' does not exist, skipping all lookups in it.
Directory '/redacted/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
======== Module name 'producer-with-package-exports/entrypoint-a' was not resolved. ========
```
