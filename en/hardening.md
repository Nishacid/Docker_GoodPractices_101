# Hardening a Docker Contaniner

## 1. Docker Daemon Exposed

You need to create a specific user for docker usage, without any other permissions, and **DON'T USE** this user for anything other docker.

Don't give the **docker** group at any user unless the specific user.

Same inside a container if you'r using docker inside docker.

***

## 2. Configure the user inside the container 

By defaut, the user created by docker is `root`, but if you setup a webserver the service will run as root. Imagine an attacker managed to get a access to you container, he will have root permissions inside.
So don't forget good practices outside containers inside !

***

## 3. Docker Capabilities and Privileges 

Every container must be started without any capabilities with the option **--cap-drop=ALL**

If a container need to be stard with a absolutely required capability, he must be configured like that : 
> **--cap-drop=ALL** --cap-add{"capability"}

Here is a non-exhaustive list of sensive capabilities among 35 avaliables : 

    - CAP_AUDIT_CONTROL
    - CAP_CHOWN
    - CAP_DAC_OVERRIDE
    - CAP_FOWNER
    - CAP_FSETID
    - CAP_IPC_OWNER
    - CAP_MKNOD
    - CAP_SETFCAP
    - CAP_SETPCAP
    - CAP_SETUID
    - CAP_SYS_ADMIN
    - CAP_SYS_CHROOT
    - CAP_SYS_BOOT
    - CAP_SYS_MODULE
    - CAP_SYS_PTRACE
    - CAP_SYS_RAWIO
    - CAP_SYS_TTY_CONFIG  


## 3.1 Isolate sensitive file systems from the host

Each Container must be booted without sharing the host's sensitive file systems: the `root (/)`, `device (/dev)`, `process (/proc)` or `virtual (/sys) file systems`. 
Therefore, the **--privileged** option should NOT be used.

***

## 4. Restrict access to host devices

If a Container is to be started with a host device, the host device must be added to the Container with the **--device** option and the additional `rwm` options specified to limit access to the minimum necessary.

***

## 5. Network 

## 5.1 Disallow container connection to docker0 bridge network

The Docker service must be configured to start a container without connecting it, by default, to the `network bridge`, `docker0`, with the option **--bridge=none**.

## 5.2 Isolate the network interface of the host

Each container must be started without sharing the network interface of the host. The option **--network host** must NOT be used.

## 5.3 Create a dedicated network for each connection

If the Container is to `expose` a port on the host's network interface or is to be connected to one or more other Containers, a dedicated network must be created for each connection.

## 6. Namespaces 

Containers uses 3 components of the **Linux Kernel** for isolate containers from one another: 
- Namespaces
- Cgroups
- OverlayFS

Namespaces essentially segregate system resources such as processes, files and memory away from other namespaces.

Every process running on Linux will be assigned two things:

- A namespace
- A process identifier (PID)

## 6.1 Dedicate PID, IPC and UTS namespaces for each container

Each Container must be started with PID, IPC and UTS spacenames separate from the host and other containers and therefore without the options **--pid**, **--ipc** and **--uts**

## 6.2 Dedicate a USER ID namespace for each container

The Docker service should be configured to start all containers with a `USER ID` separate from the host and other containers with the **--usernsremap** option. This configuration should be accompanied by a prohibition on creating new `USERID` namespace inside the container, for example, with the default `Seccomp` profile that sets a security policy to disable this feature inside the container.

## 6.3 Restrict the creation of USER ID namespace to the root user

On Debian-based Linux systems, the system configuration `kernel.unprivileged_userns_clone` must be equal to `0` (default value) to restrict the creation of a `USER ID` space to the root user (UID=0).

## 6.4 Restrict sharing of the host's USER ID namespace

When a container started sharing the `USER ID` with the host, and the Docker service is configured to start all containers with a `USER ID` separate from the host, it is possible to use the **--userns=host** option when starting the container. In this case, the threat of the host's root user (UID=0) being compromised by the Container should be considered.

***

## 7. Restricting access to resources

## 7.1 Dedicate control groups for each container

Each container must be started with `Control Groups` separate from the host and therefore without the **--cgroup-parent** option.

## 7.2 Limit host memory usage for each container

Each Container must be started with a maximum `host memory` usage limit, with the **--memory** option, and a maximum host `SWAP memory` usage limit with the **--memory-swap** option.

## 7.3 Limit host CPU usage for each container

Each Container must be started with a maximum `host CPU` usage limit with the **--cpus** option, or with the **--cpu-period** and **--cpu-quota** options.

## 7.4 Restrict read access to the root file system of each container

Each container must be started with its `rootfilesystem`, `read-only` with the **--read-only** option.

## 7.5 Limit the writing of the storage space of each container

A container can be started with a maximum limit on the use of the `host's disk space` for its `root file system`, for reading and writing with the **--storage-opt** option.

# 7.6 Limit writing to the storage space of all containers

A container can be started with its `root file system` for `read and write`, as long as Docker's local storage area, by default `/var/lib/docker/`, is a dedicated partition separate from the host's partitions.
In this case, the threat of a `denial of service` from one container to another is to be considered.

***

## 8. Container logs

## 8.1 Exporting logs with Docker

The Docker service must be configured to collect event logs from the containerized application and export them to a centralized server with the **--log-driver=\<logging driver>** option

## 8.2 Export logs from inside the container

The export of event logs from the containerized application can be implemented inside the container by a separate solution from Docker.

***

## 9. Containers Security Improvements

## 9.1 Seccomp

Secure computing mode (`seccomp`) is a Linux kernel feature. You can use it to restrict the actions available within the container. The `seccomp()` system call operates on the seccomp state of the calling process. You can use this feature to restrict your application’s access.

This feature is available only if Docker has been built with `seccomp` and the kernel is configured with `CONFIG_SECCOMP` enabled.

For example, we can deny the container the ability to perform actions such as using the mount namespace or any of the [Linux system calls](https://filippo.io/linux-syscall-table/).

You can see more [here](https://docs.docker.com/engine/security/seccomp/).

***

## 9.2 AppArmor

`**AppArmor` is a kernel enhancement to confine programs to a limited set of resources. It's a Mandatory Access Control or `MAC` that binds **access control** attributes to programs rather than to users. \
AppArmor confinement is provided via **profiles loaded into the kernel**, typically on boot. \

AppArmor profiles can be in one of **two modes**: 
- `Enforcement` Profiles loaded in enforcement mode will result in enforcement of the policy defined in the profile as well as reporting policy violation attempts (either via syslog or auditd).
- `Complain` Profiles in complain mode will not enforce policy but instead report policy violation attempts.

AppArmor differs from some other MAC systems on Linux: it is `path-based` it allows mixing of enforcement and complain mode profiles, it uses include files to ease development, and it has a far lower barrier to entry than other popular MAC systems.

***

