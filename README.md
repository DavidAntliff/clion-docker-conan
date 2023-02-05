# Docker Containers with CLion

## Remote Docker Workflow

Uses Docker to create a "remote" system, which is then treated like any remote system (sync, build & run via SSH).

https://blog.jetbrains.com/clion/2020/01/using-docker-with-clion/

This is a variation of the CLion "Remote with local sources" workflow:

https://www.jetbrains.com/help/clion/remote-projects-support.html

For this workflow, the Docker container must be built and run ahead of time, as CLion treats it as a remote host.

## Build

```
$ docker build -t clion/remote-cpp-env:0.5 -f Dockerfile.remote-cpp-env .
```

## Run

```
$ docker run -d --cap-add sys_ptrace -p127.0.0.1:2222:22 --name clion_remote_env clion/remote-cpp-env:0.5
```

## Docker Toolchain Workflow

https://www.jetbrains.com/help/clion/clion-toolchains-in-docker.html

This workflow is for local development, and uses a local Docker image to host a container for build, run and debug.

It is newer than the Remote Workflow above, and avoids redundant source synchronisation.

