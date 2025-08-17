# CrackNxNestjsDocker

## Context
```
nx version: 21.4.0
pnpm version: 10.13.1
```

## Commands used:
```
> pnpx create-nx-workspace
> nx add @nx/nest
> nx g @nx/nest:app apps/api
```

## Steps to glory
1. Running `nx serve api`
    1. Failed for the first time - timeout. 
    2. Running the second time did not start the NX Daemon - "NX Daemon is not running. Node process will not restart automatically after file changes." Ran the command `nx reset`
    3. Running the serve command for the third time started the Daemon and the app. However, on a simple change to the response of `/api` failed the webpack build.
        1. Moved the `app.controller.ts` and `app.module.ts` to the app root, and deleted the `api/src/app` and `dist` folders. Hoping for the a successful rebuild. No Luck!
        2. Ran the command `nx reset` followed by `nx server api` - rebuilt successfully. However, on file change webpack-cli build fails - can't load webpack config; complaining "Cannot read properties of undefined (reading 'data')". After further investigation, I discovered that the nx project graph was missing details for "nodes" 
            - `/.nx/workspace-data/project-graph.json` contained `nodes: {}`. It is expected to have the all project configs in the pattern 
                ```
                nodes: {
                    "@{org}/{project}": {
                        "name": string,
                        "type": string,
                        "data": Record<string, any>,
                        ...
                    }
                }
                ```
            - Hence, the error "Cannot read properties of undefined (reading 'data')"
        3. Resolution:
            - `nx reset`
            - Delete `.nx`, `node_modules`, `*/dist`, `pnpm-lock.yaml`
            - `pnpm install`
            - `nx serve api` (finally worked)


