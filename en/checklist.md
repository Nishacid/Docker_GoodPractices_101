# Docker configuration checklist

- 1. Inspect the image before usage [1. Docker image analysis](escape.md) 

# Docker configuration checklist

- Inspect the Docker image before usage, isn't impossible the image is backdoored or malicious.

- Do not use the `--privileged` flag or mount a Docker socket inside the container. The docker socket allows for spawning containers, so it is an easy way to take full control of the host, for example, by running another container with the `--privileged` flag.

- Do not run as root inside the container. Use a different user or user namespaces. The root in the container is the same as on host unless remapped with user namespaces. It is only lightly restricted by, primarily, Linux namespaces, capabilities, and cgroups.

- Drop all capabilities (`--cap-drop=all`) and enable only those that are required (`--cap-add=...`). Many of workloads don’t need any capabilities and adding them increases the scope of a potential attack.

- Use the “no-new-privileges” security option to prevent processes from gaining more privileges, for example through suid binaries.

- Limit resources available to the container. Resource limits can protect the machine from denial of service attacks.

- Adjust seccomp, AppArmor profiles to restrict the actions and syscalls available for the container to the minimum required.

- Regularly rebuild your images to apply security patches.