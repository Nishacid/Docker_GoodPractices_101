# Docker Escape Stylecheat

## 1. Docker image analysis

Imagine that we have download a image and we want to run it without any check.

<img src="./img/reverse_shell.png" alt="reverse_shell">

We can see an attacker can simply get a shell on your container.

[Dive](https://github.com/wagoodman/dive) is a tool for doing a reverse engineering of a Docker image. 

We need to download image before analysis, then take the image ID as argument for dive.

```
╰─➤  docker images                                                           
REPOSITORY                 TAG             IMAGE ID       CREATED         SIZE
mydockerimage              latest          00571c260cae   2 minutes ago   108MB


╰─➤  dive 00571c260cae
```

<img src="./img/dive.png" alt="dive">

### 1. Layers (red)
    This window shows the various layers and stages on the image, information such as the ID of the layer and any command executed in the container.

### 2. Image details (blue)
    Many information and details about the image such as size.

### 3. Current Layer Contents (green)
    All contents of the container's filesystem at the selected layer.

You can naviguate between windows using "Tab" and "Up" and "Down" for data.

<img src="./img/dive_malicious.png" alt="dive_malicious">

Take a malicious image for the example, here we can see at the 7th layer the command for the reverse shell. \
So don't forget to check all layers before running an image !

***

## 2. Docker Daemon Exposed 

### <u>First case : we are outside a container</u>

Imagine that a attacker have managed to gain a shell on the machine, but we have well (almost) configured the user right and he can't do anything.
But, if the daemon is exposed and the user have the docker group : 


```
hellodocker@ubuntu:~$ find / -name docker.sock 2>/dev/null
/run/docker.sock
hellodocker@ubuntu:~$ groups 
hellodocker docker
```

<img src="./img/privesc_outside.png">

This command essentially mounting the host "/" directory to the "/mnt" directory in a new container, chrooting and then connecting via a shell. A simple command that give a root access at the attacker. We can use any image you want, a local image or download an image if there is a internet connection.

<br>

### <u>Second case : we are inside a container</u>

The idea is the same as previously, if the user inside the container have a docker daemon exposed, we can escape the container.

> docker run -v /:/mnt --rm -it alpine chroot /mnt sh

And we have a shell on the host machine, we've breakout of the container.

### <u> Hardening </u> 

[How can I protect my container about that](./hardening.md)

***

## 3. Misconfigured Privileges and Capabilities

Docker containers allow to start without the root access but with some privileges and capabilities. \
By default, containers start with theses capabilities, who aren't necessary and which should not be used in prod :

  - CAP_AUDIT_WRITE 
  - CAP_CHOWN
  - CAP_DAC_OVERRIDE
  - CAP_FOWNER
  - CAP_FSETID
  - CAP_KILL
  - CAP_NET_BIND_SERVICE
  - CAP_NET_RAW
  - CAP_MKNOD
  - CAP_SETFC
  - CAP_SETCAP
  - CAP_SETUID/CAP_SETGID
  - CAP_SYS_CHROOT
  
You can list capabilities inside a container by using `capsh --print` (apt install libcap2-bin if doesn't exist)

The most dangerous capability is `CAP_SYS_ADMIN` because you will be able to mount files from the host OS into the container.

<img src="./img/cap_sys_admin.png" alt="cap_sys_admin">

A container would be vulnerable to this technique if run with the flags: 

> --security-opt apparmor=unconfined --cap-add=SYS_ADMIN

### <u> Hardening </u> 

[How can I protect my container about that](./hardening.md)

## 3.1 I own Root

Well configured docker containers won't allow command like fdisk -l. However on missconfigured docker command where the flag `--privileged` is specified, it is possible to get the privileges to see the host drive.

<img src="./img/fdisk.png" alt="fdisk">

We can see **/dev/sda5** who's the host partition.

<img src="./img/iownroot.png" alt="iownroot">

We can now read file from host 

## 4. Shared Namespaces

TODO

## Bonus : How to determining if you are in a container

### <u>Method 1 : processes </u>

Container often have very little processes running, run a `ps faux` to see them.

<img src="./img/processes.png" alt="processes">

### <u>Method 2 : dockerenv </u>

Docker containers allow environnements variables provides by the host, using a **.dockerenv** file. \
You can find this file in **/**. This file is created even the host don't provide any env variable.

<img src="./img/dockerenv.png" alt="dockerenv">

### <u>Method 3 : cgroup </u>

Cgroups are used by containers such as Docker. Find them in **/proc/1/cgroup**.

<img src="./img/cgroup.png" alt="cgroup">



## Automatisation 

You can launch scripts to see if you'r vulnerable to docker escape or other misconfiguration.

## Inside container : 

- [Deepce](https://github.com/stealthcopter/deepce/)
  Docker Enumeration, Escalation of Privileges and Container Escapes (DEEPCE)

- [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) 
  Local Enumeration for Privileges Escalation, with a dedicated part for docker containers. \
  EX : 
```
═════════════════════════════════════════╣ Containers ╠══════════════════════════════════════════
[+] Is this a container? ........... docker
[+] Container related tools present
[+] Any running containers? ........ No
[+] Am I inside Docker group ....... No
[+] Looking and enumerating Docker Sockets
[+] Docker version ................. Not Found
[+] Vulnerable to CVE-2019-5736 .... Not Found
[+] Vulnerable to CVE-2019-13139 ... Not Found
[+] Rooless Docker? ................ No

[+] Container & breakout enumeration
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation/docker-breakout
[+] Container ID ................... docker.escape
[+] Container Full ID .............. e061eb6346e7386a054548e1c5639f3742b35f9356937b946f2f5da8779aba4f
[+] Vulnerable to CVE-2019-5021 .. No

[+] Container Capabilities
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read+eip
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Securebits: 00/0x0/1'b0
 secure-noroot: no (unlocked)
 secure-no-suid-fixup: no (unlocked)
 secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)

[+] Privilege Mode is disabled
```

## Outside Container 

- [Docker-Bench-Security](https://github.com/docker/docker-bench-security) : The Docker Bench for Security is a script that checks for dozens of common best-practices around deploying Docker containers in production. 