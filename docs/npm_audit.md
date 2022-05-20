# NPM Audit strategy
[NPM Audit](https://docs.npmjs.com/auditing-package-dependencies-for-security-vulnerabilities) is a feature provided by the npm team. This allows to identify anomalies in a package.json/package-lock.json.

Under the hood we use [@npmcli/arborist](https://github.com/npm/arborist#readme) to fetch vulnerabilities (directly as JSON).

```js
const { vulnerabilities } = (await arborist.audit()).toJSON();
```

This strategy doesn't require the synchronization of a local database.

> ⚠️ This strategy currently only work with a local project analysis (with a package.json/package-lock.json) ⚠️

```js
import * as vuln from "@nodesecure/vuln";
import { loadRegistryURLFromLocalSystem } from "@nodesecure/npm-registry-sdk";

// Before walking the dependency tree (at runtime)
loadRegistryURLFromLocalSystem();

const dependencies = new Map();
// ...do work on dependencies...

const definition = await vuln.setStrategy(vuln.strategies.NPM_AUDIT);
await definition.hydratePayloadDependencies(dependencies, {
  // path where we have to run npm audit (default equal to process.cwd())
  path: process.cwd();
});
```

Note that it is important to call `loadRegistryURLFromLocalSystem` before running `hydratePayloadDependencies` method. The internal method will retrieve the correct URL for the registry (could be useful if the developer use a private registry for example).

## Audit a specific manifest 

For audit a specific manifest (package.json, lock-file or nodes_modules), there is the getVulnerabilities function that takes the path of the manifest and returns the vulnerabilities.

Same as `hydratePayloadDependencies` Under the hood we use @npmcli/arborist to fetch vulnerabilities (directly as JSON).

```js
/**
 * @param {string} path
 * @return Promise<{ [keys: string]: any }[]> | TypeError
 */
async function getVulnerabilities(path) {
  const arborist = new Arborist({ ...NPM_TOKEN, path });

  try {
    return (await arborist.audit()).toJSON().vulnerabilities;
  }
  catch (error) {
    return error;
  }
}
```

Example:
```js
import * as vuln from "@nodesecure/vuln";

const definition = await vuln.setStrategy(vuln.strategies.NPM_AUDIT);
const vulnerabilites = await definition.getVulnerabilities('./package.json');
```
