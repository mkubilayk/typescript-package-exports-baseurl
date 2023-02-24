Title: Using `baseUrl` breaks package exports lookup

# Bug Report

A producer package exposes multiple entrypoint using [Node's package exports](https://nodejs.org/api/packages.html#package-entry-points). When this package is consumed using the following compiler options, TypeScript fails to resolve the entry points.

```jsonc
{
    "compilerOptions": {
        "module": "commonjs",
        "moduleResolution": "node16",
        "baseUrl": "/path/to/modules/store",
    }
}
```

This problem doesn't happen when `baseUrl` is absent -- relying on the default lookup behavior.

### 🔎 Search Terms

`baseUrl`, base url, package exports, module resolution


### 🕗 Version & Regression Information

This is the behavior in every version I tried: 4.7, 4.8 and 4.9. I was unable to test this on prior versions because `"node16"` module resolution was first introduced in 4.7.

### ⏯ Playground Link

http://github.com/mkubilayk/typescript-package-exports-baseurl

### 💻 Code

Please see the full code and reproduction steps in the repo but the most relevant bits are as follows.

Producer:

```jsonc
{
    "name": "producer-with-package-exports",
    "version": "1.0.0",
    "main": "out/index.js",
    "types": "out/index.d.ts",
    "exports": {
        ".": {
            "types": "./out/index.d.ts",
            "require": "./out/index.js"
        },
        "./entrypoint-a": {
            "types": "./out/entrypoint-a.d.ts",
            "require": "./out/entrypoint-a.js"
        }
    }
}
```

Consumer (targetting `"commonjs"`):

```ts
import { something } from "producer-with-package-exports";
import { somethingElse } from "producer-with-package-exports/entrypoint-a";
//                            ^
// Cannot find module 'producer-with-package-exports/entrypoint-a' or its corresponding type declarations. ts(2307)
```

### 🙁 Actual behavior

When using the `baseUrl` compiler option in the consumer `"producer-with-package-exports/entrypoint-a"` fails to resolve:

```ts
import { somethingElse } from "producer-with-package-exports/entrypoint-a";
//                            ^
// Cannot find module 'producer-with-package-exports/entrypoint-a' or its corresponding type declarations. ts(2307)
```

### 🙂 Expected behavior

Using `baseUrl` should not stop TypeScript from looking at package exports in the producer. So `"producer-with-package-exports/entrypoint-a"` should be resolved correctly -- matching the behaviour when `baseUrl` is not present.

---

I have also verified the producer package using Andrew Branch's https://arethetypeswrong.github.io (💙) and got the following report as expected:

<img width="785" alt="image" src="https://user-images.githubusercontent.com/2494233/221135106-cb0f2704-8d4a-4174-84c0-2746b4999e0c.png">

<details>
<summary>Expand to see the full report</summary>

```jsonc
{
  "packageName": "producer-with-package-exports",
  "containsTypes": true,
  "entrypointResolutions": {
    ".": {
      "node10": {
        "name": ".",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.d.ts",
          "isJson": false,
          "isTypeScript": true
        },
        "implementationResolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.js",
          "isJson": false,
          "isTypeScript": false
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Node10'.",
          "Loading module 'producer-with-package-exports' from 'node_modules' folder, target file types: TypeScript, Declaration.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "File '/node_modules/producer-with-package-exports.ts' does not exist.",
          "File '/node_modules/producer-with-package-exports.tsx' does not exist.",
          "File '/node_modules/producer-with-package-exports.d.ts' does not exist.",
          "'package.json' does not have a 'typesVersions' field.",
          "'package.json' does not have a 'typings' field.",
          "'package.json' has 'types' field 'out/index.d.ts' that references '/node_modules/producer-with-package-exports/out/index.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/index.d.ts' exists - use it as a name resolution result.",
          "======== Module name 'producer-with-package-exports' was successfully resolved to '/node_modules/producer-with-package-exports/out/index.d.ts' with Package ID 'producer-with-package-exports/out/index.d.ts@1.0.0'. ========"
        ]
      },
      "node16-cjs": {
        "name": ".",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.d.ts",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": true
        },
        "implementationResolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.js",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": false
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Node16'.",
          "Resolving in CJS mode with conditions 'node', 'require', 'types'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath '.' with target './out/index.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/index.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports' was successfully resolved to '/node_modules/producer-with-package-exports/out/index.d.ts' with Package ID 'producer-with-package-exports/out/index.d.ts@1.0.0'. ========"
        ]
      },
      "node16-esm": {
        "name": ".",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.d.ts",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": true
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports' from '/index.mts'. ========",
          "Explicitly specified module resolution kind: 'Node16'.",
          "Resolving in ESM mode with conditions 'node', 'import', 'types'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath '.' with target './out/index.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/index.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports' was successfully resolved to '/node_modules/producer-with-package-exports/out/index.d.ts' with Package ID 'producer-with-package-exports/out/index.d.ts@1.0.0'. ========"
        ]
      },
      "bundler": {
        "name": ".",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/index.d.ts",
          "isJson": false,
          "isTypeScript": true
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Bundler'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath '.' with target './out/index.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/index.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports' was successfully resolved to '/node_modules/producer-with-package-exports/out/index.d.ts' with Package ID 'producer-with-package-exports/out/index.d.ts@1.0.0'. ========"
        ]
      }
    },
    "./entrypoint-a": {
      "node10": {
        "name": "./entrypoint-a",
        "trace": [
          "======== Resolving module 'producer-with-package-exports/entrypoint-a' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Node10'.",
          "Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file types: TypeScript, Declaration.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "'package.json' does not have a 'typesVersions' field.",
          "File '/node_modules/producer-with-package-exports/entrypoint-a.ts' does not exist.",
          "File '/node_modules/producer-with-package-exports/entrypoint-a.tsx' does not exist.",
          "File '/node_modules/producer-with-package-exports/entrypoint-a.d.ts' does not exist.",
          "'package.json' does not have a 'typings' field.",
          "'package.json' has 'types' field 'out/index.d.ts' that references '/node_modules/producer-with-package-exports/entrypoint-a/out/index.d.ts'.",
          "Loading module as file / folder, candidate module location '/node_modules/producer-with-package-exports/entrypoint-a/out/index.d.ts', target file types: TypeScript, Declaration.",
          "File name '/node_modules/producer-with-package-exports/entrypoint-a/out/index.d.ts' has a '.d.ts' extension - stripping it.",
          "Directory '/node_modules/@types' does not exist, skipping all lookups in it.",
          "Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file types: JavaScript, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "'package.json' does not have a 'typesVersions' field.",
          "File '/node_modules/producer-with-package-exports/entrypoint-a.js' does not exist.",
          "File '/node_modules/producer-with-package-exports/entrypoint-a.jsx' does not exist.",
          "'package.json' has 'main' field 'out/index.js' that references '/node_modules/producer-with-package-exports/entrypoint-a/out/index.js'.",
          "Loading module as file / folder, candidate module location '/node_modules/producer-with-package-exports/entrypoint-a/out/index.js', target file types: JavaScript, JSON.",
          "File name '/node_modules/producer-with-package-exports/entrypoint-a/out/index.js' has a '.js' extension - stripping it.",
          "======== Module name 'producer-with-package-exports/entrypoint-a' was not resolved. ========"
        ]
      },
      "node16-cjs": {
        "name": "./entrypoint-a",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": true
        },
        "implementationResolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/entrypoint-a.js",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": false
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports/entrypoint-a' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Node16'.",
          "Resolving in CJS mode with conditions 'node', 'require', 'types'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath './entrypoint-a' with target './out/entrypoint-a.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports/entrypoint-a' was successfully resolved to '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' with Package ID 'producer-with-package-exports/out/entrypoint-a.d.ts@1.0.0'. ========"
        ]
      },
      "node16-esm": {
        "name": "./entrypoint-a",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts",
          "moduleKind": 1,
          "isJson": false,
          "isTypeScript": true
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports/entrypoint-a' from '/index.mts'. ========",
          "Explicitly specified module resolution kind: 'Node16'.",
          "Resolving in ESM mode with conditions 'node', 'import', 'types'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath './entrypoint-a' with target './out/entrypoint-a.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports/entrypoint-a' was successfully resolved to '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' with Package ID 'producer-with-package-exports/out/entrypoint-a.d.ts@1.0.0'. ========"
        ]
      },
      "bundler": {
        "name": "./entrypoint-a",
        "resolution": {
          "fileName": "/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts",
          "isJson": false,
          "isTypeScript": true
        },
        "trace": [
          "======== Resolving module 'producer-with-package-exports/entrypoint-a' from '/index.ts'. ========",
          "Explicitly specified module resolution kind: 'Bundler'.",
          "File '/package.json' does not exist.",
          "Loading module 'producer-with-package-exports/entrypoint-a' from 'node_modules' folder, target file types: TypeScript, JavaScript, Declaration, JSON.",
          "Found 'package.json' at '/node_modules/producer-with-package-exports/package.json'.",
          "Entering conditional exports.",
          "Matched 'exports' condition 'types'.",
          "Using 'exports' subpath './entrypoint-a' with target './out/entrypoint-a.d.ts'.",
          "File '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' exists - use it as a name resolution result.",
          "Resolved under condition 'types'.",
          "Exiting conditional exports.",
          "======== Module name 'producer-with-package-exports/entrypoint-a' was successfully resolved to '/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts' with Package ID 'producer-with-package-exports/out/entrypoint-a.d.ts@1.0.0'. ========"
        ]
      }
    }
  },
  "fileExports": {
    "/node_modules/producer-with-package-exports/out/index.d.ts": {
      "something": {
        "name": "something",
        "flags": 2,
        "valueDeclarationRange": [
          21,
          35
        ]
      }
    },
    "/node_modules/producer-with-package-exports/out/index.js": {
      "__esModule": {
        "name": "__esModule",
        "flags": 1048580,
        "valueDeclarationRange": [
          14,
          75
        ]
      },
      "something": {
        "name": "something",
        "flags": 1048580,
        "valueDeclarationRange": [
          105,
          122
        ]
      }
    },
    "/node_modules/producer-with-package-exports/out/entrypoint-a.d.ts": {
      "somethingElse": {
        "name": "somethingElse",
        "flags": 2,
        "valueDeclarationRange": [
          21,
          39
        ]
      }
    },
    "/node_modules/producer-with-package-exports/out/entrypoint-a.js": {
      "__esModule": {
        "name": "__esModule",
        "flags": 1048580,
        "valueDeclarationRange": [
          14,
          75
        ]
      },
      "somethingElse": {
        "name": "somethingElse",
        "flags": 1048580,
        "valueDeclarationRange": [
          109,
          130
        ]
      }
    }
  }
}
```
</details>
