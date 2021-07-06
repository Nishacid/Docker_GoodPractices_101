# Hardening a Docker Contaniner

## Docker Daemon Exposed

You need to create a specific user for docker usage, without any other permissions, and **DON'T USE** this user for anything other docker.

Don't give the **docker** group at any user unless the specific user.

Same inside a container if you'r using docker inside docker.

## Configure the user inside the container 

By defaut, the user created by docker is `root`, but if you setup a webserver the service will run as root. Imagine an attacker managed to get a access to you container, he will have root permissions inside.
So don't forget good practices outside containers inside !


## Docker Seccomp 

Secure computing mode (`seccomp`) is a Linux kernel feature. You can use it to restrict the actions available within the container. The `seccomp()` system call operates on the seccomp state of the calling process. You can use this feature to restrict your application’s access.

This feature is available only if Docker has been built with `seccomp` and the kernel is configured with `CONFIG_SECCOMP` enabled.

For example, we can deny the container the ability to perform actions such as using the mount namespace or any of the [Linux system calls](https://filippo.io/linux-syscall-table/).