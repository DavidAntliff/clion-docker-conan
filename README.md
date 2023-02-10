# Docker Containers with CLion & Conan

There are at least two ways to use a Docker build environment with CLion:

1. Remote Docker workflow - where the Docker container is treated in the same way as a remote host,
2. Docker Toolchain workflow - where CLion natively supports spawning local containers to run build tasks.

The Docker Toolchain workflow is the newer of the two. However, it creates some complications beyond simple CMake
projects because the containers that CLion uses to execute the build are ephemeral. Thankfully the generated build
files are stored locally, and are not lost when the containers are closed. Therefore, with some manual intervention,
it is possible to configure additional build facilities such as Conan.

## Remote Docker Workflow

This workflow is included for completeness. It is not explored further.

Uses Docker to create a "remote" system, which is then treated like any remote system (sync, build & run via SSH).

https://blog.jetbrains.com/clion/2020/01/using-docker-with-clion/

This is a variation of the CLion "Remote with local sources" workflow:

https://www.jetbrains.com/help/clion/remote-projects-support.html

For this workflow, the Docker container must be built and run ahead of time, as CLion treats it as a remote host.

### Build

```
$ docker build -t clion/remote-cpp-env:0.5 -f Dockerfile.remote-cpp-env .
```

### Run

```
$ docker run -d --cap-add sys_ptrace -p127.0.0.1:2222:22 --name clion_remote_env clion/remote-cpp-env:0.5
```

CLion can be configured as a typical remote toolchain, with SSH credentials to connect to the running container.

Obviously, if you stop the container, CLion is unable to use it.

## Docker Toolchain Workflow

https://www.jetbrains.com/help/clion/clion-toolchains-in-docker.html

This workflow is for local development, and uses a local Docker image to host a container for build, run and debug.

It is newer than the Remote Workflow above, and avoids redundant source synchronisation. CLion will manage the
life-cycles of the containers it spawns to perform build tasks.

A basic CMake project is trivial to set up, but it becomes more complicated when additional components are added
to the project, for example Conan package management. In this case, manual intervention of the container seems to be
required. See `Dockerfile` for Conan-specific additions.

### Build

Before CLion can use a Docker image for this workflow, it needs to be built:

```
$ docker build --build-arg UID=$(id -u) -t clion/ubuntu/cpp-env:1.0 -f Dockerfile.cpp-env-ubuntu .
```

### Run

When running, bind-mount the local Conan cache directory, to reuse it:

```
$ docker run --rm -it -v /Users/david/.conan/data:/home/builder/.conan/data clion/ubuntu/cpp-env:1.0
```

This volume binding should also be added to CLion's Docker toolchain config, as a Volume Binding under Container Settings.

### Conan

A profile called `gcc-11-debug` is created by the Dockerfile, and can be used with Conan.

Create the `conanfile.txt` in your project and use the `CMakeDeps` and `CMakeToolchain` generators. Then mount the
dev container with the local source mounted as `/tmp/myproject` (to match what CLion uses):

```
$ cd myproject
$ docker run --rm -it \
    -v /Users/david/.conan/data:/home/builder/.conan/data \
    -v $(pwd):/tmp/myproject \
    clion/ubuntu/cpp-env:1.0

# whatever CLion has called the build directory;
$ cd /tmp/myproject/cmake-build-debug-dockercpp-env-ubuntu10
$ conan install --build=missing -pr gcc-11-debug ..
```

This will create `conan_toolchain.cmake` in the build directory, which can now be used to build with:

```
$ cmake .. -DCMAKE_TOOLCHAIN_FILE=conan_toolchain.cmake -DCMAKE_BUILD_TYPE=Debug
```

Add `-DCMAKE_TOOLCHAIN_FILE=conan_toolchain.cmake` to CLion's CMake > CMake options field.

At this point, the CLion project should now configure, build, debug and run successfully.
