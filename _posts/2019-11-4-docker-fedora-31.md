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

You are not alone, this is the first release that comes with cgroup v2 enabled by default and Docker still don't support cogrup v2.

To use Docker you have to rollback to cgroup v1, reboot the kernel with `systemd.unified_cgroup_hierarchy=0`:

```
# dnf install -y grubby
# sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
```



---
[1]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-list> "GET followers/list"
[2]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-followers-ids> "GET followers/ids"
[3]: <https://developer.twitter.com/en/docs/accounts-and-users/follow-search-get-users/api-reference/get-users-lookup> "GET users/lookup"
[4]: <https://metacpan.org/pod/Twitter::API> "Twitter::API - A Twitter REST API library for Perl"
[5]: <https://github.com/someone-stole-my-name/Twitter_followers_examples> "Examples repository"