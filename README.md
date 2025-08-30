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
## TLDR: 
- **Use WSL2**
    - Do not mount windows file system. Move the code to the distro, instead.
- Add the packages needed by the rust-based native watcher to the dockerfile - `bash, git, inotify-tools, libc6-dev, and build-essential`
- Mount the app directory - I mounted `/apps/api/src` to avoid mounting the `/dist`, `node_modules`, and `.nx` folders.

## Steps to glory
1. Added Dockerfile (linux-based base image) and docker-compose.yml with local repository mounted (excluding node_modules and nx cache directory)
    - The daemon ran but the hot reload did not function. Code changes propagated correctly to the container but webpack did not rebuild the project. Updating the code from within the container (docker exec) did not restart the server either.
2. NX suggests running the Daemon locally and share the socket directory and NX cache (mount them) with the container services.
    - Doing so resulted in infinite reloads.
    - Also, sharing the socket directory did not make sense to me since the Linux and Windows deals with sockets differently.
    - Perhaps, NX's documentation was written assuming their users always use Linux machines
2. Tried out webpack polling. Multiple sources suggest using webpack polling (`watchOptions: {poll: 1000}`), so that webpack would rebuilt the project on file changes. This strays away from relying on NX. 
    - Although that is not the goal of this project, I still gave it a try. But still, no luck!
3. Read NX's code. This pushed me to think that the Daemon was not listening for file changes, eventhough it was running. 
    - After going through NX's code base, I learned that NX uses a rust-based native file watcher. It is responsible for detecting file changes and notifying the Daemon.
    - After a bit more research it was evident that the base image I used for the Dockerfile, did not have the necessary packages to run the rust-based native watcher.
4. Added the necessary packages required by the rust-based native watcher - bash, git, inotify-tools, libc6-dev, and build-essential
    - Ran docker compose without a mount, and updated the code fron within the container (docker exec). The watcher picked up the changes and restarted the server!
5. Mounted the local repository to the service. This time changing the file did not trigger a server restart. 
    - Researching a bit more, I realized that the issue is mounting Windows directory to a Linux system results in `inotify` (what the native watcher relies on for detecting file changes) not detecting changes. This is a well documented limitation of docker. 
    - This issue is not evident when using webpack polling - a method recommended by nestJS for implemented HMR. One can conainerize the webpack-polling enabled application with Docker and mount the local directory and expect the HMR to work seamlessly. But this is not true with NX.
6. Switched to WSL2 (Ubuntu distro). Cut all ties to Windows filesytem and moved the codebase to WSL2 (Ubuntu distro with node v22 and pnpm v10.13.1). The hot reload worked seamlessly with the local reporsitory mounted - updating code locally reload the service in container immediately. 





