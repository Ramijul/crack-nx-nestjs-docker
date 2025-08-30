# CrackNxNestjsDocker

## Context
```
nx version: 21.4.0
pnpm version: 10.13.1
os: Windows 11
```

## Commands used:
```
> pnpx create-nx-workspace
> nx add @nx/nest
> nx g @nx/nest:app apps/api
```

## Goal
Nx does a wonderful job at hot reloading on file changes. However, if we want to run a database locally for our projects, it would be an inconvinence to run the database and our applications separately. A better DX would be to use `docker compose` to spin up all the services we need and allow hot reloading within the running services. 

Nx's hot reloading seems to not work when containerzed, even though the aforementioned version of Nx allows configuring it run the Daemon in a container. 

An easy solution would be to simply run the applications with Docker Compose as we would in any other monorepo (take yarn workspaces, for instance). This means not utilizing Nx tasks management tool, defeating the purpose of using Nx.

Therefore, the purpose of the project is to understand how Nx works and finding a way to containerize it. Note, that their documentation is unclear on the shortcomings.

## Approach to the problem
The project is initialized with a nestJS application `api`. First step is to containerize the entire Nx project and getting it to run in a container with hot relaod enabled. Second, Add a second application to the Nx project, and containerize that aswell, while running the Daemon in a separate container and letting the other services/apps communicate with it.

- The branch `running-native-watcher-in-docker` documents the first step. Successfully, running the entire project in docker container with hot relaod.
- The branch `shared-daemon-container` documents the 2nd step. Running the daemon separately and share it with other services running.

At every branch, the README.md contains the documentation on the problems faced and reasoning behind the appraoch taken.

## Side note on Running Nx locally (without docker)
    1. `nx serve api` Failed for the first time - timeout error. 
    2. Running the second time did not start the NX Daemon - "NX Daemon is not running. Node process will not restart automatically after file changes."
    3. Looks like adding the env `NX_DAEMON = true` is required for the Daemon to run. Although, NX's documentation does not suggest so.
    4. Running `nx serve api` started the Daemon and enable hot relaod.


