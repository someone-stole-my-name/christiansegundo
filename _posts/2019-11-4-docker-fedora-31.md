---
layout: post
title: Docker errors on Fedora 31
category: Gnu-Linux
tags:
- docker
- fedora
- cgroups
---

If you are facing theese issues with Docker on Fedora 31:

```
docker: Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused "process_linux.go:275: applying cgroup configuration for process caused "open /sys/fs/cgroup/cpuset/docker/cpuset.cpus: no such file or directory"": unknown.
```

```
dockerd[10141]: Error starting daemon: Devices cgroup isn’t mounted
systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
```

You are not alone, this is the first release that comes with cgroup v2 enabled by default and Docker still don't support cgroup v2.

To use Docker you have to rollback to cgroup v1, reboot the kernel with `systemd.unified_cgroup_hierarchy=0`:

```
# dnf install -y grubby
# sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
```

Or use Podman which is a (sort of) drop-in replacement<sup>[1]</sup> for Docker that already supports cgroup v2.

---
[1]: <https://media.ccc.de/v/ASG2018-177-replacing_docker_with_podman#t=3> "Replacing Docker with Podman"