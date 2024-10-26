---
title: Threat Hunting in Docker Uncovering Weaknesses
description: Explore Docker misconfigurations attackers exploit, like exposed APIs and privileged containers, and learn detection tips for threat hunters
date: 2023-11-27
image: dockerr.webp
license: 
comments: false
draft: false

categories:
    - Threat hunting
    - Docker
---




Hi 👋, in this post, we’ll explore common Docker misconfigurations attackers exploit to compromise systems, such as exposed APIs and privileged containers, with detection strategies for threat hunters.

---

## Exposed Docker APIs

Exposing Docker's API to the internet without authentication allows attackers full control over containers.

### Exploitation
Attackers use tools like **Shodan** to find exposed APIs on ports **2375** or **2376** and create containers remotely:

```bash
curl -X POST http://<docker_host_ip>:2375/containers/create -d '{"Image":"alpine"}'
```

### Detection
1. Use **Nmap** to scan for open Docker API ports:

```bash
nmap -p 2375 --open <target_subnet>
```

2. Monitor API logs for requests from external IPs.

---

## Privileged Containers

Running containers in **privileged mode** gives attackers root-level access to the host.

### Exploitation
An attacker inside a privileged container can mount the host's root directory:

```bash
docker run --privileged -v /:/host --rm -it alpine chroot /host
```

### Detection
Identify privileged containers:

```bash
docker inspect --format '{{ .HostConfig.Privileged }}' $(docker ps -q)
```

---

## Insecure Volume Mounts

Improper mounts of sensitive directories (like `/etc` or `/root`) give attackers access to critical host data.

### Exploitation
Attackers can mount the host file system:

```bash
docker run -v /:/host --rm -it alpine /bin/sh
cat /host/etc/shadow
```

### Detection
Check for dangerous mounts:

```bash
docker inspect --format '{{ .Mounts }}' $(docker ps -q)
```

---

## Cgroups and Namespace Escapes

Excessive privileges, like `SYS_ADMIN`, can break Docker's isolation.

### Exploitation
Run a container with the `SYS_ADMIN` capability to escape to the host:

```bash
docker run --cap-add=SYS_ADMIN -it alpine /bin/sh
```

### Detection
Inspect running containers for dangerous capabilities:

```bash
docker inspect --format '{{ .HostConfig.CapAdd }}' $(docker ps -q)
```

---

## Resource Exhaustion: Denial of Service

Containers without resource limits can overwhelm system resources, leading to Denial of Service (DoS) attacks.

### Exploitation
Run a container to consume excessive CPU:

```bash
docker run --rm -it --cpu-shares=1024 --memory=1g stress --cpu 8 --timeout 60
```

### Detection
Monitor resource usage with `docker stats`:

```bash
docker stats
```

---

**References**:

- [Docker Security Best Practices](https://docs.docker.com/engine/security/security/)
- [Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

