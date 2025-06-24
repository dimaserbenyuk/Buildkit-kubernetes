# Buildkit-kubernetes
Buildkit-kubernetes

```shell
brew install buildkit
```

1. without ssl

apply

```shell
kubectl apply -f buildkit-no-ssl-arm.yml
```

port-forward

```shell
kubectl port-forward service/buildkitd-arm 1234:1234 -n buildkit
```
```shell
Forwarding from 127.0.0.1:1234 -> 1234
Forwarding from [::1]:1234 -> 1234
Handling connection for 1234
```

check workers

```shell
buildctl --addr tcp://127.0.0.1:1234 debug workers
```
output, all good 

```shell
ID				PLATFORMS
vvgzvgoj6iexoir7096hz07fm	linux/arm64,linux/amd64,linux/amd64/v2,linux/riscv64,linux/ppc64le,linux/s390x,linux/386,linux/arm/v7,linux/arm/v6

```
info

```shell
buildctl --addr tcp://127.0.0.1:1234 debug info
```

output

```shell
BuildKit: github.com/moby/buildkit v0.23.0 cc8ff80e5733eb0a0347176009232d6e40752f7f
```

create remote builder

```shell
docker buildx create --name remote --driver remote tcp://127.0.0.1:1234 --use
```
output
```shell
remote
```

check buildx

```shell
docker buildx ls
```

output

```shell
NAME/NODE           DRIVER/ENDPOINT                                                                                     STATUS     BUILDKIT   PLATFORMS
kube                kubernetes
 \_ kube0            \_ kubernetes:///kube?deployment=buildkit-132097b2-0aa3-422f-a13d-49c728de008d-f5389&kubeconfig=   inactive
remote*             remote
 \_ remote0          \_ tcp://127.0.0.1:1234                                                                            running    v0.23.0    linux/amd64 (+2), linux/arm64, linux/arm (+2), linux/ppc64le, (3 more)
default             docker
 \_ default          \_ default                                                                                         running    v0.21.0    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
desktop-linux       docker
 \_ desktop-linux    \_ desktop-linux                                                                                   running    v0.21.0    linux/amd64 (+2), linux/arm64, linux/ppc64le, linux/s390x, (2 more)
```

buildx

```shell
remote*             remote
 \_ remote0          \_ tcp://127.0.0.1:1234
```

docker run without cache

```shell
docker buildx build \
    --platform linux/arm64 \
    --builder=remote \
    -t django:latest \
    -f Dockerfile . \
    --progress=plain \
    --load
```

<details>
<summary>Docker running logs</summary>
<br>
building with "remote" instance using remote driver.
<br><br>
<pre>
#0 building with "remote" instance using remote driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 1.92kB done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/python:3.13-slim
#2 DONE 1.0s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/9] FROM docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81
#4 resolve docker.io/library/python:3.13-slim@sha256:f2fdaec50160418e0c2867ba3e254755edd067171725886d5d303fd7057bbf81 done
#4 DONE 0.0s

#5 [2/9] WORKDIR /usr/src/app
#5 CACHED

#6 [internal] load build context
#6 transferring context: 1.56kB done
#6 DONE 0.0s

#7 [3/9] RUN apt-get update && apt-get install -y --no-install-recommends     build-essential     libpq-dev     gcc     libcairo2     libpango-1.0-0     libpangocairo-1.0-0     libgdk-pixbuf-2.0-0     libffi-dev     shared-mime-info     libxml2     libxslt1.1     libjpeg-dev     libglib2.0-0     fonts-liberation  && apt-get clean && rm -rf /var/lib/apt/lists/*
#7 0.668 Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
#7 1.048 Get:2 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
#7 1.196 Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
#7 1.326 Get:4 http://deb.debian.org/debian bookworm/main arm64 Packages [8693 kB]
#7 2.103 Get:5 http://deb.debian.org/debian bookworm-updates/main arm64 Packages [756 B]
#7 2.227 Get:6 http://deb.debian.org/debian-security bookworm-security/main arm64 Packages [264 kB]
#7 2.609 Fetched 9213 kB in 2s (3704 kB/s)
...
...
#13 DONE 0.1s

#14 exporting to oci image format
#14 exporting layers
#14 exporting layers 77.4s done
#14 exporting manifest sha256:212eaf6eacaaf8eb5e90ca10dc048445ece74a4326d4e8f204401f2e6aae1193
#14 exporting manifest sha256:212eaf6eacaaf8eb5e90ca10dc048445ece74a4326d4e8f204401f2e6aae1193 done
#14 exporting config sha256:bdaea0e8ef6da32d24cdd1eed008cc4c769a022490edf7dacd1dc61bccaf7c9e done
#14 sending tarball
#14 ...

#15 importing to docker
#15 DONE 0.0s

#14 exporting to oci image format
#14 sending tarball 4.7s done
#14 DONE 82.1s
</pre>
</details>

connect to pod check cache

```shell
ls -lah /var/lib/buildkit
```
output

```shell
/ 
total 356K   
drwxr-xr-x    3 root     root        4.0K Jun 24 10:20 .
drwxr-xr-x    1 root     root        4.0K Jun 24 10:20 ..
-rw-------    1 root     root           0 Jun 24 10:20 buildkitd.lock
-rw-------    1 root     root      128.0K Jun 24 10:36 cache.db
-rw-------    1 root     root      256.0K Jun 24 10:38 history.db
drwx------    6 root     root        4.0K Jun 18 15:15 runc-overlayfs
```

